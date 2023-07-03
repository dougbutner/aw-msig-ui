<script lang="ts">
    import { writable } from 'svelte/store'
    import { onMount } from 'svelte/internal'
    import { Session, SessionKit } from '@wharfkit/session'
    import { WebRenderer } from '@wharfkit/web-renderer'
    import { WalletPluginAnchor } from '@wharfkit/wallet-plugin-anchor'
    import { WalletPluginCloudWallet } from '@wharfkit/wallet-plugin-cloudwallet'
    import { Transaction, Action, TimePointSec, Name, PermissionLevel } from '@wharfkit/antelope';
    import { Input, Select, ChoiceInput, Divider, Button } from "agnostic-svelte";

    const ui = new WebRenderer()
    const session = writable<Session | null>(null)

    let dao_name = '', proposal_title = '', proposal_description = '', account = '', name = '', abi = null, action_data = [], action_values = {}, proposal_id = '';
    const planets = [
        { value: 'eyeke', label: 'Eyeke' },
        { value: 'kavian', label: 'Kavian' },
        { value: 'magor', label: 'Magor' },
        { value: 'naron', label: 'Naron' },
        { value: 'nerix', label: 'Neri' },
        { value: 'veles', label: 'Veles' },
    ];

    const sessionKit = new SessionKit({
        appName: 'aw-msig-ui',
        chains: [
            {
                id: '1064487b3cd1a897ce03ae5b6a865651747e2e152090f99c1d19d44e01aea5a4',
                url: 'https://wax.greymass.com',
                explorer: {
                    prefix: 'https://waxblock.io/transaction/',
                    suffix: '',
                }
            }
        ],
        ui,
        walletPlugins: [
            new WalletPluginAnchor(),
            new WalletPluginCloudWallet(),
        ]
    })

    onMount(async () => {
        try {
            const restored = await sessionKit.restore()
            console.log(restored)
            if (restored) {
                session.set(restored)
            }
        } catch (e) {
            console.log('error caught in onMount', e)
        }
    })

    async function login() {
        const response = await sessionKit.login()
        $session = response.session
    }

    async function logout() {
        await sessionKit.logout($session)
        $session = null
    }

    function getRandProposalName() {
        const str = '12345abcdefghijklmnopqrstuvwxyz';
        let name = '';
        while (name.length < 12) {
            const rnd = Math.round(Math.random() * str.length);
            name += str.substring(rnd, rnd + 1);
        }
        return name;
    }

    function getDAOAccount(dao_name) {
        if (dao_name === 'nerix'){
            return 'neri.dac';
        }
        else {
            return `${dao_name}.dac`;
        }
    }

    async function getRequested(dao_name) {
        const requested = [];
        requested.push({actor: getDAOAccount(dao_name), permission: 'active'});
        return requested;
    }

    async function loadABI(){
        abi = await $session.abiProvider.getAbi(Name.from(account));
        abi.actions_select = abi.actions.map(a => {
            return {value: a.name, label: a.name};
        });
        console.log(abi.actions_select);
    }

    async function loadActionData() {
        if (abi && name){
            const type = abi.structs.find((s) => {
                return (s.name === name);
            });
            console.log(type);
            action_data = type.fields;
        }
    }

    async function propose() {
        const proposalName = getRandProposalName();
        const requested = await getRequested(dao_name[0]);
        for (let k in action_values){
            if (action_values[k].substring(0, 1) === '['){
                action_values[k] = JSON.parse(action_values[k]);
            }
        }
        console.log('action values', action_values);
        const act = Action.from({
            account,
            name,
            authorization: [PermissionLevel.from(`${getDAOAccount(dao_name[0])}@active`)],
            data: action_values
        }, abi);
        const trx = Transaction.from({
            context_free_actions: [],
            actions: [act],
            ref_block_num: 0,
            ref_block_prefix: 0,
            expiration: TimePointSec.fromMilliseconds((new Date()).getTime() + (60 * 60 * 24 * 7 * 1000))
        });
        const action = {
            account: 'msig.worlds',
            name: 'propose',
            authorization: [$session.permissionLevel],
            data: {
                proposer: $session.actor,
                proposal_name: proposalName,
                requested,
                dac_id: dao_name[0],
                metadata: [{key: 'title', value: proposal_title}, {key: 'description', value: proposal_description}],
                trx
            },
        }
        console.log('propose action action', action);

        $session.transact({ action }, { broadcast: true }).catch((e) => {
            console.log('error caught in propose', e)
        })
    }

    async function approve() {
        const action = {
            account: 'msig.worlds',
            name: 'approve',
            authorization: [$session.permissionLevel],
            data: {
                dac_id: dao_name[0],
                level: $session.permissionLevel,
                proposal_name: proposal_id
            },
        }
        console.log('approve action action', action);

        $session.transact({ action }, { broadcast: true }).catch((e) => {
            console.log('error caught in approve', e)
        })
    }
</script>

{#if $session}
                <ChoiceInput id="planetID" name="planetName" size="large" type="radio" isInline options={planets} legendLabel="Planet" bind:checked={dao_name} />{dao_name}

                <Input label="Proposal Title" bind:value={proposal_title} placeholder="Enter a short name for the proposal" />
                <Input type="textarea" label="Proposal Description" placeholder="Enter a long description which can help the custodians when making a decision" bind:value={proposal_description} />

            <Divider size="large" />

                <Input bind:value={account} label="Contract" placeholder="Contract Name" on:blur={loadABI} />
                {#if abi}
                <div>
                    <Select
                            uniqueId="sel3"
                            name="select3"
                            size="large"
                            labelCopy="Action"
                            defaultOptionLabel="Select action name"
                            options={abi.actions_select}
                            on:selected={(e) => {name = e.detail; loadActionData()} }
                            bind:selected={name}
                    />
                </div>
                {/if}

                {#if action_data.length}
                    {#each action_data as ad}
                        <Input bind:value={action_values[ad.name]} label="{ad.name}" name="action_data[{ad.name}]" placeholder="{ad.type}" />
                    {/each}
                {/if}

    <div id="propose_btn">
        <Button mode="primary" on:click={propose}>
            Propose
        </Button>
    </div>

    <div>
        <h2>Approve Proposal</h2>
        <Input label="Proposal Name" bind:value={proposal_id} placeholder="Enter the proposal ID" />
        <div id="approve_btn">
            <Button mode="primary" on:click={approve}>
                Approve
            </Button>
        </div>
    </div>

    <!--<button on:click={logout}> Logout </button>-->
{:else}
    <Button mode="primary" on:click={login}> Login </Button>
{/if}

<style>
    input[type=text], textarea {
        font-size: 1.1em;
        width: 60%;
    }
    textarea {
        height: 6em;
    }
    form > div {
        padding: 0.2em;
    }
    #propose_btn, #approve_btn {
        padding-top: 1.5em;
    }
</style>