<script>
    import { each } from "svelte/internal";
    import ListItem from "./ListItem.svelte";
    import { createEventDispatcher } from "svelte";
    import { onMount } from "svelte";
    import axios from "axios";

    export let chat_list;
    export let user_data;

    const dispatch = createEventDispatcher();
    const host = "http://127.0.0.1:8000";
    var originalUsers = [];
    var users = [];

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
        let fileInput = document.getElementById("file_input");
        const file = fileInput.files[0];
        const formData = new FormData();
        formData.append("file", file);

        await axios
            .post(
                `${host}/monchat/profile_upload/${user_data.user_name}/`,
                formData,
                {
                    headers: {
                        "Content-Type": "multipart/form-data",
                    },
                }
            )
            .then((response) => {
                axios
                    .get(`${host}/monchat/user/${user_data.user_id}/`, {
                        headers: {
                            "Content-Type": "multipart/form-data",
                        },
                    })
                    .then((response) => {
                        user_data = response.data.data;
                        document.getElementById(
                            "profile-bg"
                        ).style.backgroundImage = `url(${host}/media/${user_data.user_icon}/)`;
                    });
            })
            .catch((err) => {
                alert(err);
            });
    }

    let uploadedFile = [];
    $: if (uploadedFile.length > 0) {
        upload();
    }

    async function getUsers() {
        await axios
            .get(`${host}/monchat/users/${user_data.user_id}/`, {
                headers: {
                    "Content-Type": "application/json",
                },
            })
            .then((response) => {
                users = response.data.data;
                originalUsers = users;
            });
    }

    // onMount(() => {
    //     let fileBtn = document.getElementById("file_input");
    //     let profileBtn = document.getElementById("profile-bg");
    //     profileBtn.addEventListener("click", () => {
    //         fileBtn.click();
    //     });
    // });

    var sidebar = "chats";
    var mutated = false;

    $: if (sidebar === "profile") {
        if (mutated) {
            axios
                .get(`${host}/monchat/user/${user_data.user_id}/`, {
                    headers: { "Content-Type": "application/json" },
                })
                .then((response) => {
                    user_data = response.data.data;
                });
        }

        setTimeout(() => {
            let fileBtn = document.getElementById("file_input");
            let profileBtn = document.getElementById("profile-bg");
            profileBtn.addEventListener("click", () => {
                fileBtn.click();
            });
            document.getElementById("fname").focus();
        }, 2000);
    } else if (sidebar === "chats" && mutated) {
        let payload = {
            fname: document.getElementById("fname").value,
            lname: document.getElementById("lname").value,
            user_bio: document.getElementById("bio").value,
        };
        axios
            .put(`${host}/monchat/user/${user_data.user_id}/`, payload, {
                headers: { "Content-Type": "application/json" },
            })
            .then((response) => {
                // alert(response.data.msg);
            })
            .catch((err) => {
                alert(err);
            });
    } else {
        getUsers();
    }

    var userSearchQ;

    const execUserSearch = (q) => {
        let result = [];
        q = q.toLowerCase();
        console.log(q);
        originalUsers.forEach((user) => {
            if (
                user.first_name.toLowerCase().includes(q) ||
                user.last_name.toLowerCase().includes(q)
            ) {
                result.push(user);
            }
        });
        users = result;
    };

    $: if (userSearchQ) {
        execUserSearch(userSearchQ);
    } else {
        users = originalUsers;
    }
</script>

<div class="list-container">
    {#if sidebar === "profile"}
        <div class="profile-page">
            <div class="side-topbar">
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <span class="back-arrow" on:click={(e) => (sidebar = "chats")}>
                    <svg
                        viewBox="0 0 24 24"
                        height="24"
                        width="24"
                        preserveAspectRatio="xMidYMid meet"
                        class=""
                        version="1.1"
                        x="0px"
                        y="0px"
                        enable-background="new 0 0 24 24"
                        xml:space="preserve"
                        ><path
                            fill="currentColor"
                            d="M12,4l1.4,1.4L7.8,11H20v2H7.8l5.6,5.6L12,20l-8-8L12,4z"
                        /></svg
                    >
                </span>
            </div>
            <div class="profile" id="profile">
                <h3>Your Profile</h3>
                <div
                    id="profile-bg"
                    style="background-image: url({host}/media/{user_data.user_icon}/);width: 200px;height: 200px;"
                />
                <span class="img-overlay" id="upload_btn">
                    <svg
                        viewBox="0 0 24 24"
                        height="24"
                        width="24"
                        preserveAspectRatio="xMidYMid meet"
                        class=""
                        version="1.1"
                        x="0px"
                        y="0px"
                        enable-background="new 0 0 24 24"
                        xml:space="preserve"
                    >
                        <path
                            fill="currentColor"
                            d="M21.317,4.381H10.971L9.078,2.45C8.832,2.199,8.342,1.993,7.989,1.993H4.905 c-0.352,0-0.837,0.211-1.078,0.468L1.201,5.272C0.96,5.529,0.763,6.028,0.763,6.38v1.878c0,0.003-0.002,0.007-0.002,0.01v11.189 c0,1.061,0.86,1.921,1.921,1.921h18.634c1.061,0,1.921-0.86,1.921-1.921V6.302C23.238,5.241,22.378,4.381,21.317,4.381z  M12.076,18.51c-3.08,0-5.577-2.497-5.577-5.577s2.497-5.577,5.577-5.577s5.577,2.497,5.577,5.577 C17.654,16.013,15.157,18.51,12.076,18.51z M12.076,9.004c-2.17,0-3.929,1.759-3.929,3.929s1.759,3.929,3.929,3.929 s3.929-1.759,3.929-3.929C16.004,10.763,14.245,9.004,12.076,9.004z"
                        />
                    </svg>
                    <br />
                    <span>Change Profile Picture</span>
                </span>
                <input
                    type="file"
                    id="file_input"
                    bind:files={uploadedFile}
                    hidden
                />
                <div class="profile-input">
                    <span class="legend">First Name</span>
                    <input
                        id="fname"
                        type="text"
                        placeholder="First Name"
                        value={user_data.first_name}
                        on:change={(e) => (mutated = true)}
                    />
                    <br />
                    <span class="legend">Last Name</span>
                    <input
                        id="lname"
                        type="text"
                        placeholder="Last Name"
                        value={user_data.last_name}
                        on:change={(e) => (mutated = true)}
                    />
                    <span class="legend">Your Bio</span>
                    <textarea
                        id="bio"
                        class="bio"
                        type="text"
                        placeholder="Your Bio"
                        value={user_data.user_bio}
                        on:change={(e) => (mutated = true)}
                    />
                </div>
            </div>
        </div>
    {:else if sidebar == "chats"}
        <div id="chats">
            <div class="side-topbar">
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <img
                    class="profile-img"
                    id="profile_btn"
                    src={`${host}/media/${user_data.user_icon}/`}
                    alt={"profile"}
                    on:click={(e) => (sidebar = "profile")}
                />
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
                    <ListItem
                        {...chat}
                        clickCallback={(e) => handleChatClick(chat)}
                    />
                {/each}
            {:else}
                <span class="empty"
                    ><i>No recent chats, kindly start a chat.</i></span
                >
            {/if}

            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <span
                class="new-contact"
                on:click={(e) => {
                    sidebar = "users";
                }}
            >
                <svg
                    viewBox="0 0 24 24"
                    height="24"
                    width="24"
                    preserveAspectRatio="xMidYMid meet"
                    class=""
                    version="1.1"
                    x="0px"
                    y="0px"
                    enable-background="new 0 0 24 24"
                    xml:space="preserve"
                    ><path
                        fill="currentColor"
                        enable-background="new    "
                        d="M19.005,3.175H4.674C3.642,3.175,3,3.789,3,4.821V21.02 l3.544-3.514h12.461c1.033,0,2.064-1.06,2.064-2.093V4.821C21.068,3.789,20.037,3.175,19.005,3.175z M14.016,13.044H7.041V11.1 h6.975V13.044z M17.016,9.044H7.041V7.1h9.975V9.044z"
                    /></svg
                >
            </span>
        </div>
    {:else if sidebar == "users"}
        <div id="user_list" class="users">
            <div class="side-topbar">
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <span class="back-arrow" on:click={(e) => (sidebar = "chats")}>
                    <svg
                        viewBox="0 0 24 24"
                        height="24"
                        width="24"
                        preserveAspectRatio="xMidYMid meet"
                        class=""
                        version="1.1"
                        x="0px"
                        y="0px"
                        enable-background="new 0 0 24 24"
                        xml:space="preserve"
                        ><path
                            fill="currentColor"
                            d="M12,4l1.4,1.4L7.8,11H20v2H7.8l5.6,5.6L12,20l-8-8L12,4z"
                        /></svg
                    >
                </span>
            </div>
            <input
                id="user_search"
                type="text"
                placeholder="Search for a user"
                bind:value={userSearchQ}
            />
            <ul>
                {#each users as user, index}
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <li
                        on:click={(e) => {
                            dispatch("newchat", user);
                        }}
                        title={user.user_bio}
                    >
                        <img
                            src={`${host}/media/${user.user_icon}/`}
                            alt={user.user_id}
                        />
                        <span class="name"
                            >{`${user.first_name} ${user.last_name}`}</span
                        >
                        <span class="bio-text"
                            >{user.user_bio.length < 50
                                ? user.user_bio
                                : `${user.user_bio.slice(0, 50)}...`}</span
                        >
                    </li>
                {/each}
            </ul>
        </div>
    {/if}
</div>

<style>
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

    .back-arrow svg {
        color: azure;
        cursor: pointer;
        margin: 12px;
    }

    .profile-page {
        height: 100%;
        width: 100%;
        color: azure;
    }

    .profile-input {
        margin-top: 40px;
    }
    .profile-input input {
        width: 70%;
        text-align: center;
        margin-bottom: 24px;
        margin-left: 15%;
        background-color: #202c33;
        border-radius: 9px;
        border: none;
        color: #f0ffff;
        outline: none;
    }
    .profile-input .bio {
        width: 70%;
        height: 200px;
        border: none;
        color: #f0ffff;
        margin-left: 15%;
        outline: none;
        text-align: center;
        background-color: #202c33;
        border-radius: 9px;
    }

    .profile-input .legend {
        margin-left: 16%;
        color: #00a884;
    }

    .profile {
        padding-bottom: 160px;
    }

    .profile h3 {
        text-align: center;
    }

    #profile-bg {
        top: 0;
        border-radius: 50%;
        margin-left: 110px;
        margin-top: 30px;
        cursor: pointer;
        background-size: 200px;
        background-repeat: no-repeat;
        background-position: center;
    }

    .img-overlay {
        border-radius: 50%;
        background-color: rgba(90, 90, 90, 0.6);
        padding: 10px;
        width: 180px;
        height: 180px;
        margin-left: 110px;
        margin-top: -200px;
        color: azure;
        display: none;
        text-align: center;
        position: absolute;
    }

    #profile-bg:hover + .img-overlay {
        display: block;
    }

    .img-overlay svg {
        width: 40px;
        height: 40px;
        margin-top: 30px;
    }
    .img-overlay span {
        width: 60px;
    }

    .users {
        width: 100%;
        height: 100%;
        padding: 10px 0;
        color: azure;
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
        width: 50px;
        height: 50px;
        margin: 5px;
    }

    .users li .name {
        text-align: center;
        padding: 2px;
        margin-top: 12px;
        left: 75px;
        font-weight: bold;
        position: absolute;
    }

    .users li .bio-text {
        left: 75px;
        font-size: 12px;
        margin-top: 35px;
        text-align: center;
        position: absolute;
    }

    .users li:hover {
        background-color: #202c33;
    }

    #user_search {
        margin-left: 26px;
        background-color: #202c338c;
        border: none;
        outline: none;
        color: azure;
        border-radius: 14px;
        width: 85%;
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
</style>
