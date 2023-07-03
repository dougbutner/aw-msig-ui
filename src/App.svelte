<script lang="ts">
    import Login from './lib/Login.svelte'
    import 'agnostic-svelte/css/common.min.css';
    import awLogo from './assets/AW_community_logo.svg'
</script>

<main>
    <div>
        <a href="https://alienworlds.io" target="_blank" rel="noreferrer">
            <img src={awLogo} class="logo aw" alt="Alien Worlds" />
        </a>
    </div>
    <h1>Custom Proposal Generator</h1>
    <p>Use this form to create a proposal for a planet, you must know the contract, action and parameters for the action
        you want to call.  The resulting proposal will execute this action with the permissions of the planet's .dac
        account (active permission only)</p>

    <div class="card">
        <Login />
    </div>

</main>

<style>
    h1 {
        font-size: 2.4em;
    }
    .logo {
        height: 8em;
        padding: 1.5em;
        will-change: filter;
        transition: filter 300ms;
    }
    .logo.aw:hover {
        filter: drop-shadow(0 0 2em #333333aa);
    }
</style>
