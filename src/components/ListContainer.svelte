<script>
    import { each } from "svelte/internal";
    import ListItem from "./ListItem.svelte";
    import { createEventDispatcher } from "svelte";
    import { onMount } from "svelte";
    import axios from "axios";

    export let chat_list;
    export let profile_img;
    export let user_name;

    var users = [];

    const dispatch = createEventDispatcher();
    const host = "http://127.0.0.1:8000";

    function handleChatClick(chat) {
        dispatch("chatclick", {
            ...chat,
        });
    }

    function hide_menu(e) {
        let drop_down = document.getElementById("menu_dropdown");
        let drop_down_list = document.getElementById("menu_list");

        let not_found = true;
        drop_down_list.childNodes.forEach((node) => {
            console.log(node === e.target);
            if (node === e.target) {
                not_found = false;
            }
        });

        if (not_found) {
            drop_down.style.opacity = "0";
            setTimeout((e) => {
                drop_down.style.display = "none";
            }, 1200);
            setTimeout((e) => {
                console.log("removing lst");
                document.removeEventListener("click", hide_menu);
            }, 500);
        }
    }

    function show_menu(e) {
        let drop_down = document.getElementById("menu_dropdown");
        drop_down.style.opacity = "1";
        drop_down.style.display = "block";

        setTimeout((e) => {
            console.log("listerner set");
            document.addEventListener("click", hide_menu);
        }, 500);
    }

    function logout(e) {
        window.localStorage.setItem("monchat_user_id", "");
        window.location.reload();
    }

    function showSettings(e) {
        // settings page
    }

    async function upload() {
        console.log("Preparing file upload...");
        let fileInput = document.getElementById("file_input");
        const file = fileInput.files[0];
        const formData = new FormData();
        formData.append("file", file);

        await axios
            .post(`${host}/monchat/profile_upload/${user_name}/`, formData, {
                headers: {
                    "Content-Type": "multipart/form-data",
                },
            })
            .then((response) => {
                alert(response.data.msg);
            })
            .catch((err) => {
                alert(err);
            });
    }

    let uploadedFile = [];
    $: if (uploadedFile.length > 0) {
        upload();
    }

    function check_nodes(e) {
        let user_list_nodes =
            document.getElementById("user_list").lastChild.childNodes;
        let found_node = false;
        user_list_nodes.forEach((node) => {
            if (node == e.target) {
                found_node = true;
            }
        });

        if (!found_node) {
            document.getElementById("user_list").style.display = "none";
        }
    }

    async function display_users() {
        await axios
            .get(`${host}/monchat/users/`, {
                headers: {
                    "Content-Type": "application/json",
                },
            })
            .then((response) => {
                users = response.data.data;

                document.getElementById("user_list").style.display = "block";
                document.addEventListener("click", check_nodes);
            });
    }

    onMount(() => {
        let fileBtn = document.getElementById("file_input");
        let profileBtn = document.getElementById("profile_btn");
        profileBtn.addEventListener("click", () => {
            fileBtn.click();
        });
    });
</script>

<div class="list-container">
    <div class="side-topbar">
        <img
            class="profile-img"
            id="profile_btn"
            src={`${host}/media/${profile_img}/`}
            alt={"profile"}
        />
        <input type="file" id="file_input" bind:files={uploadedFile} hidden />
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <span class="dot-menu" on:click={show_menu}>
            <svg
                viewBox="0 0 24 24"
                height="24"
                width="24"
                preserveAspectRatio="xMidYMid meet"
                class="chat-menu"
                version="1.1"
                x="0px"
                y="0px"
                enable-background="new 0 0 24 24"
                xml:space="preserve"
            >
                <path
                    fill="currentColor"
                    d="M12,7c1.104,0,2-0.896,2-2c0-1.105-0.895-2-2-2c-1.104,0-2,0.894-2,2 C10,6.105,10.895,7,12,7z M12,9c-1.104,0-2,0.894-2,2c0,1.104,0.895,2,2,2c1.104,0,2-0.896,2-2C13.999,9.895,13.104,9,12,9z M12,15 c-1.104,0-2,0.894-2,2c0,1.104,0.895,2,2,2c1.104,0,2-0.896,2-2C13.999,15.894,13.104,15,12,15z"
                />
            </svg>
        </span>
    </div>
    <div class="menu-dropdown" id="menu_dropdown">
        <ul id="menu_list">
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <li on:click={showSettings}>Settings</li>
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <li on:click={logout}>Logout</li>
        </ul>
    </div>
    <input
        type="text"
        class="chat-search"
        placeholder="Search or start a new chat"
    />
    <svg
        class="filter"
        viewBox="0 0 24 24"
        height="22"
        width="22"
        preserveAspectRatio="xMidYMid meet"
        version="1.1"
        x="0px"
        y="0px"
        enable-background="new 0 0 24 24"
        xml:space="preserve"
    >
        <path
            fill="currentColor"
            d="M10,18.1h4v-2h-4V18.1z M3,6.1v2h18v-2H3z M6,13.1h12v-2H6V13.1z"
        />
    </svg>
    {#if chat_list.length > 0}
        {#each chat_list as chat}
            <ListItem {...chat} clickCallback={(e) => handleChatClick(chat)} />
        {/each}
    {:else}
        <span class="empty"><i>No recent chats, kindly start a chat.</i></span>
    {/if}

    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span class="new-contact" on:click={display_users}>+</span>
    <div id="user_list" class="users">
        <input id="user_search" type="text" placeholder="Search for a user" />
        <ul>
            {#each users as user, index}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <li
                    on:click={(e) => {
                        dispatch("newchat", user);
                    }}
                >
                    <img
                        src={`${host}/media/${user.user_icon}/`}
                        alt={user.user_id}
                    />
                    <span>{`${user.first_name} ${user.last_name}`}</span>
                </li>
            {/each}
        </ul>
    </div>
</div>

<style>
    .menu-dropdown {
        width: 210px;
        padding: 10px 0;
        background-color: #202c33;
        border-radius: 6px;
        margin: 0;
        position: absolute;
        box-shadow: -2px 5px 4px 1px rgba(0, 0, 0, 0.482);
        left: 45%;
        top: 5px;
        color: azure;
        z-index: 10;
        display: none;
        transition: opacity 1s ease;
    }

    .users {
        width: 300px;
        padding: 10px 0;
        color: azure;
        position: fixed;
        left: 28%;
        bottom: 45px;
        background-color: rgba(240, 255, 255, 0.2);
        backdrop-filter: blur(4px);
        box-shadow: 2px 3px 5px 2px rgba(0, 0, 0, 0.482);
        border-radius: 8px;
        display: none;
        border-bottom-left-radius: 0;
        border-bottom-right-radius: 0;
    }

    .users ul {
        margin: 0;
        padding: 0;
    }

    .users li {
        margin: 0;
        padding: 5px;
        cursor: pointer;
        display: flex;
        flex-wrap: wrap;
        list-style-type: none;
    }

    .users li img {
        border-radius: 50%;
        width: 40px;
        height: 40px;
        margin: 5px;
    }

    .users li span {
        text-align: center;
        padding: 2px;
        margin: 0;
        margin-top: 10px;
        left: 60px;
        position: absolute;
    }

    .users li:hover {
        background-color: #111b21;
    }

    #user_search {
        margin-left: 26px;
        background-color: #202c338c;
        border: none;
        outline: none;
        color: azure;
        border-radius: 14px;
    }

    .new-contact {
        background-color: #00a884;
        color: azure;
        font-size: 40px;
        font-weight: bold;
        text-align: center;
        width: 50px;
        height: 50px;
        position: fixed;
        bottom: 20px;
        left: 24%;
        cursor: pointer;
        box-shadow: 5px 3px 4px 2px rgba(0, 0, 0, 0.482);
        border-radius: 50%;
    }

    .menu-dropdown ul {
        margin: 0;
        padding: 0;
    }

    .menu-dropdown li {
        padding: 7px;
        cursor: pointer;
        list-style-type: none;
    }
    .menu-dropdown li:hover {
        background-color: #111b21;
    }
    .list-container {
        display: inline-block;
        width: 30%;
        margin: 0;
        height: 100%;
        padding: 10px 0;
        padding-bottom: 60px;
        background-color: #111b21;
        margin-top: 50px;
        position: fixed;
        overflow-y: auto;
        scrollbar-width: thin;
        scrollbar-color: rgba(240, 255, 255, 0.2) transparent;
        border-right: 0.5px solid rgba(255, 255, 255, 0.5);
    }

    .dot-menu {
        right: 8px;
        top: 12px;
        color: azure;
        cursor: pointer;
        position: absolute;
    }

    .empty {
        color: rgba(240, 255, 255, 0.5);
        margin-top: 20%;
        left: 40px;
        position: absolute;
        width: 300px;
        z-index: 4;
    }

    .side-topbar {
        border-right: 0.5px solid rgba(255, 255, 255, 0.5);
        background-color: #202c33;
        width: 30%;
        height: 50px;
        margin: 0;
        top: 0;
        right: 0;
        left: 0;
        z-index: 2;
        position: fixed;
        overflow: hidden;
    }
    .profile-img {
        width: 40px;
        height: 40px;
        margin: 5px;
        border-radius: 50%;
        margin-left: 10px;
    }

    .chat-search {
        width: 85%;
        margin-left: 10px;
        background-color: #202c33;
        border-radius: 9px;
        border: none;
        color: #f0ffff;
        outline: none;
    }

    .filter {
        color: #f0ffff80;
        margin-left: 10px;
        cursor: pointer;
    }
</style>
