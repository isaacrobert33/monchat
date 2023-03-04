<script>
    export let msg;
    export let msg_time;
    export let direction;
</script>

{#if direction==="me"}
    <div class="me-msg-card">
        <div class="my">
            <span class="tail-out">
                <svg viewBox="0 0 8 13" height="13" width="8" preserveAspectRatio="xMidYMid meet" class="" version="1.1" x="0px" y="0px" enable-background="new 0 0 8 13" xml:space="preserve"><path opacity="0.13" d="M5.188,1H0v11.193l6.467-8.625 C7.526,2.156,6.958,1,5.188,1z"></path><path fill="currentColor" d="M5.188,0H0v11.193l6.467-8.625C7.526,1.156,6.958,0,5.188,0z"></path></svg>
            </span>
            <span class="chat-msg">{msg}</span>
            <span class="chat-time">{msg_time}</span>
        </div>
    </div>
{:else}
    <div class="you-msg-card ">
        <div class="you">
            <span class="tail-in">
                <svg viewBox="0 0 8 13" height="13" width="8" preserveAspectRatio="xMidYMid meet" class="" version="1.1" x="0px" y="0px" enable-background="new 0 0 8 13" xml:space="preserve"><path opacity="0.13" fill="#0000000" d="M1.533,3.568L8,12.193V1H2.812 C1.042,1,0.474,2.156,1.533,3.568z"></path><path fill="currentColor" d="M1.533,2.568L8,11.193V0L2.812,0C1.042,0,0.474,1.156,1.533,2.568z"></path></svg>
            </span>
            <span class="chat-msg">{msg}</span>
            <span class="chat-time">{msg_time}</span>
        </div>
    </div>
{/if}



<style>
    .you-msg-card {
        width: 100%;
        margin: 10px 0;
        margin-left: 50px;
        min-height: 30px;
        display: block;
        position: relative;
    }

    .me-msg-card {
        width: 99%;
        margin: 10px 0;
        margin-right: 20px;
        min-height: 30px;
        display: block;
        position: relative;
    }

    .you {
        max-width: 40%;
        padding: 5px;
        color: azure;
        border-radius: 7.5px;
        background-color: #202c33;
        border-top-left-radius: 0;
    }

    .tail-in {
        color: #202c33;
        position: absolute;
        margin-top: -7.5px;
        margin-left: -12px;
        z-index: 1;
    }

    .chat-msg {
        font-size: 14px;
    }

    .chat-time {
        font-size: 10px;
        color: rgba(240, 255, 255, 0.5);
    }

    .my {
        right: 40px;
        padding: 5px;
        max-width: 40%;
        color: azure;
        background-color: #005c4b;
        border-radius: 7.5px;
        border-top-right-radius: 0;
        position: absolute;
    }

    .tail-out {
        color: #005c4b;
        position: absolute;
        right: -8px;
        margin-top: -7.5px;
        z-index: 1;
    }

    .chat-msg {
        font-size: 14px;
    }

    .chat-time {
        font-size: 10px;
        color: rgba(240, 255, 255, 0.5);
    }
</style>