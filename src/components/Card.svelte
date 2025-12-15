<script lang="ts">
    const { logo, style, href } = $props();
</script>

<a class="card" style={style} href={href}>
    <img class="logo" src={logo} alt="Card Logo" />
</a>

<style>
    .card {
        width: 100%;
        padding: 32px;
        border: 4px solid #47425E;
        border-radius: 16px;

        translate: 0px 0px;
        box-shadow: 4px 4px 0px rgba(0, 0, 0, 0.25);
        
        transition: all 600ms cubic-bezier(0.3, 0.7, 0.4, 1);        
    }

    .card:hover {
        translate: 4px 4px;
        box-shadow: 0px 0px 0px rgba(0, 0, 0, 0.25);

        cursor: pointer;
    }
</style>