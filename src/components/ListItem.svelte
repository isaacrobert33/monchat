<script>
    export let chat_title;
    export let chat_msg_prev;
    export let chat_icon;
</script>

<div class="list-item">
    <img src={chat_icon} alt={chat_title} />
    <div class="chat-info">
        <p class="title">{chat_title}</p>
        <p class="msg">{chat_msg_prev}</p>
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