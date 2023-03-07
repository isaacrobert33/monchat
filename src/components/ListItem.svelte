<script>
    export let msg_recipient;
    export let msg_body;
    export let clickCallback;
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="list-item" on:click={clickCallback}>
    <img src={`profile_assets/${msg_recipient.user_icon}`} alt={msg_recipient.user_name} />
    <div class="chat-info">
        <p class="title">{msg_recipient.user_name}</p>
        <p class="msg">{msg_body}</p>
    </div>
</div>

<style>
    .list-item {
        display: block;
        margin: 0;
        color: azure;
        border-bottom: 0.5px solid rgba(255, 255, 255, 0.5);
        padding: 5px;
        cursor: pointer;
    }

    .list-item:hover {
        background-color: #202c33;
    }

    .chat-info {
        display: inline-block;
        position: absolute;
        margin-left: 10px;
        margin-top: 5px;
    }

    .title {
        font-weight: bold;
        font-size: 18px;
    }

    p {
        margin: 1px;
        font-size: 13px;
    }

    img {
        width: 50px;
        height: 50px;
        display: inline-block;
    }
</style>