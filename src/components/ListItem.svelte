<script>
    export let msg_id;
    export let msg_recipient;
    export let msg_sender;
    export let msg_body;
    export let msg_time;
    export let direction;
    export let unread_count;
    export let clickCallback;

    var host = "http://127.0.0.1:8000";
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div id={msg_id} class="list-item" on:click={clickCallback}>
    <img
        src={`${host}/media/${
            direction == "outbound"
                ? msg_recipient.user_icon
                : msg_sender.user_icon
        }/`}
        alt={msg_recipient.user_name}
    />
    <div class="chat-info">
        <p class="title">
            {direction == "outbound"
                ? `${msg_recipient.first_name} ${msg_recipient.last_name}`
                : `${msg_sender.first_name} ${msg_sender.last_name}`}
        </p>
        <p class="msg">{msg_body}</p>
    </div>

    {#if unread_count}
        <span class="unread-count">{unread_count}</span>
    {/if}
    <span
        class="msg_time"
        style="color: {unread_count
            ? '#00a884'
            : '#f0ffff80'}; font-weight: {unread_count ? 'bold' : 'light'};"
        >{msg_time}</span
    >
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
        border-radius: 50%;
        display: inline-block;
    }

    .unread-count {
        border-radius: 50%;
        color: #111b21;
        background-color: #00a884;
        padding: 5px;
        width: 10px;
        height: 10px;
        text-align: center;
        margin-top: 23px;
        font-size: 12px;
        font-weight: bold;
        right: 12px;
        position: absolute;
    }

    .msg_time {
        position: absolute;
        right: 10px;
        margin-top: 5px;
        color: #f0ffff80;
        font-size: 13px;
    }
</style>
