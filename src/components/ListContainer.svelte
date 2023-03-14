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
    var users4group = [];
    var users = [];

    let originalChatList = [];
    $: if (!originalChatList.length > 0) {
        originalChatList = chat_list;
    }

    function handleChatClick(chat) {
        dispatch("chatclick", {
            ...chat,
        });
    }

    function hide_menu(e) {
        let drop_down = document.getElementById("menu_dropdown");
        let drop_down_list = document.getElementById("menu_list");

        let not_found = true;
        if (drop_down_list) {
            drop_down_list.childNodes.forEach((node) => {
                if (node === e.target) {
                    not_found = false;
                }
            });
        }

        if (not_found && drop_down) {
            drop_down.style.opacity = "0";
            setTimeout((e) => {
                drop_down.style.display = "none";
            }, 1200);
            setTimeout((e) => {
                document.removeEventListener("click", hide_menu);
            }, 500);
        } else {
            document.removeEventListener("click", hide_menu);
        }
    }

    function show_menu(e) {
        let drop_down = document.getElementById("menu_dropdown");
        drop_down.style.opacity = "1";
        drop_down.style.display = "block";

        setTimeout((e) => {
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

    var newGroupUsers = [];

    async function createNewGroup() {
        let mb = newGroupUsers.map((user) => user.user_id);
        let payload = {
            name: "",
            description: "",
            members: mb,
        };
        await axios.post(
            `${host}/monchat/group/${user_data.user_id}/`,
            payload,
            {
                headers: {
                    "Content-Type": "application/json",
                },
            }
        );
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

    async function uploadGroupIcon() {
        let fileInput = document.getElementById("group_file_input");
        const file = fileInput.files[0];
        const formData = new FormData();
        formData.append("file", file);

        await axios
            .post(
                `${host}/monchat/group_upload/${user_data.user_id}/`,
                formData,
                {
                    headers: {
                        "Content-Type": "multipart/form-data",
                    },
                }
            )
            .then((response) => {
                console.log(response);
            })
            .catch((err) => {
                alert(err);
            });
    }

    let uploadedFile = [];
    $: if (uploadedFile.length > 0) {
        upload();
    }

    let groupIcon = [];
    $: if (groupIcon.length > 0) {
        uploadGroupIcon();
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
                users4group = users;
            });
    }

    var sidebar = "new_group";
    // "chats";
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
    } else if (sidebar == "new_group") {
        let fileIn = document.getElementById("group_file_input");
        let btn = document.getElementById("group_icon_btn");
        btn.addEventListener("click", (e) => {
            fileIn.click();
        });
    } else {
        getUsers();
    }

    var usersIndex = {};
    var userSearchQ;

    const execUserSearch = (q) => {
        let result = [];
        q = q.toLowerCase();
        originalUsers.forEach((user) => {
            if (
                user.first_name.toLowerCase().includes(q) ||
                user.last_name.toLowerCase().includes(q) ||
                user.user_name.includes(q)
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

    const execChatSearch = (q) => {
        let result = [];
        q = q.toLowerCase();
        console.log(originalChatList);
        originalChatList.forEach((chat) => {
            if (
                chat.msg_body.toLowerCase().includes(q) ||
                (chat.msg_recipient.first_name.toLowerCase().includes(q) &&
                    chat.direction == "outbound") ||
                (chat.msg_recipient.last_name.toLowerCase().includes(q) &&
                    chat.direction == "outbound") ||
                (chat.msg_recipient.user_name.toLowerCase().includes(q) &&
                    chat.direction == "outbound") ||
                (chat.msg_sender.first_name.toLowerCase().includes(q) &&
                    chat.direction == "inbound") ||
                (chat.msg_sender.last_name.toLowerCase().includes(q) &&
                    chat.direction == "inbound") ||
                (chat.msg_sender.user_name.toLowerCase().includes(q) &&
                    chat.direction == "inbound")
            ) {
                result.push(chat);
            }
        });
        if (q) {
            console.log("res", result);
            chat_list = result;
        } else {
            console.log("none", originalChatList);
            chat_list = originalChatList;
        }
    };

    var filter = false;
    const filterChats = (e) => {
        document.getElementById("filter").style.backgroundColor = filter
            ? "transparent"
            : "#00a884";
        document.getElementsByClassName("filter")[0].style.color = filter
            ? "#f0ffff80"
            : "azure";
        if (filter) {
            chat_list = originalChatList;
            filter = false;
        } else {
            let unreadFilter = [];
            originalChatList.forEach((chat) => {
                if (chat.unread_count > 0) {
                    unreadFilter.push(chat);
                }
            });
            chat_list = unreadFilter;
            filter = true;
        }
    };

    let chatSearchQ = "";

    $: {
        execChatSearch(chatSearchQ);
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
                    <li
                        on:click={(e) => {
                            sidebar = "new_group_choice";
                        }}
                    >
                        New Group
                    </li>
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <li on:click={showSettings}>Settings</li>
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <li on:click={logout}>Logout</li>
                </ul>
            </div>
            <div class="search-bar">
                <svg
                    width="24px"
                    height="24px"
                    viewBox="0 0 24 24"
                    fill="#f0ffff80"
                    xmlns="http://www.w3.org/2000/svg"
                >
                    <path
                        d="M11.01 20.02C15.9861 20.02 20.02 15.9861 20.02 11.01C20.02 6.03391 15.9861 2 11.01 2C6.03391 2 2 6.03391 2 11.01C2 15.9861 6.03391 20.02 11.01 20.02Z"
                    />
                    <path
                        d="M21.9901 18.95C21.6601 18.34 20.9601 18 20.0201 18C19.3101 18 18.7001 18.29 18.3401 18.79C17.9801 19.29 17.9001 19.96 18.1201 20.63C18.5501 21.93 19.3001 22.22 19.7101 22.27C19.7701 22.28 19.8301 22.28 19.9001 22.28C20.3401 22.28 21.0201 22.09 21.6801 21.1C22.2101 20.33 22.3101 19.56 21.9901 18.95Z"
                    />
                </svg>
                <input
                    type="text"
                    class="chat-search"
                    bind:value={chatSearchQ}
                    placeholder="Search or start a new chat"
                />
            </div>
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <span id="filter" on:click={filterChats}>
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
            </span>

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
                class="user-search"
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
                            ><b>{`${user.first_name} ${user.last_name}`}</b> • {`${user.user_name}`}</span
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
    {:else if sidebar == "new_group_choice"}
        <div id="new_group" class="new-group">
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
                <h3>Add the group members</h3>
            </div>
            <div class="group-members">
                {#each newGroupUsers as user}
                    <span class="new-member">
                        <img
                            src={`${host}/media/${user.user_icon}/`}
                            alt={user.user_id}
                        />
                        <span class="member-name"
                            >{`${user.first_name} ${user.last_name}`}</span
                        >
                        <!-- svelte-ignore a11y-click-events-have-key-events -->
                        <span
                            class="member-cancel"
                            on:click={(e) => {
                                let n = [...newGroupUsers];
                                n.splice(newGroupUsers.indexOf(user), 1);
                                newGroupUsers = n;
                                users4group.splice(
                                    usersIndex[user.user_name],
                                    0,
                                    user
                                );
                                users4group = users4group;
                            }}>&times;</span
                        >
                    </span>
                {/each}
            </div>
            <input class="user-search" placeholder="Enter user's name" />
            <ul>
                {#each users4group as user, index}
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <li
                        on:click={(e) => {
                            newGroupUsers = [...newGroupUsers, user];
                            let n = users4group;
                            n.splice(users4group.indexOf(user), 1);
                            users4group = n;
                            usersIndex[user.user_name] = index;
                        }}
                        title={user.user_bio}
                    >
                        <img
                            src={`${host}/media/${user.user_icon}/`}
                            alt={user.user_id}
                        />
                        <span class="name"
                            ><b>{`${user.first_name} ${user.last_name}`}</b> • {`${user.user_name}`}</span
                        >
                        <span class="bio-text"
                            >{user.user_bio.length < 50
                                ? user.user_bio
                                : `${user.user_bio.slice(0, 50)}...`}</span
                        >
                    </li>
                {/each}
            </ul>
            {#if newGroupUsers.length > 0}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <span
                    class="group-btn"
                    on:click={(e) => {
                        sidebar = "new_group";
                    }}
                >
                    <svg
                        viewBox="0 0 30 30"
                        height="30"
                        width="30"
                        preserveAspectRatio="xMidYMid meet"
                        class=""
                        version="1.1"
                        x="0px"
                        y="0px"
                        enable-background="new 0 0 30 30"
                        xml:space="preserve"
                        ><path
                            fill="currentColor"
                            d="M15,7l-1.4,1.4l5.6,5.6H7v2h12.2l-5.6,5.6L15,23l8-8L15,7z"
                        /></svg
                    >
                </span>
            {/if}
        </div>
    {:else if sidebar == "new_group"}
        <div class="new_group_main">
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
                    <h3>New Group</h3>
                </span>
            </div>

            <span class="group-bg">
                <svg
                    fill="#f0ffff"
                    viewBox="0 0 1920 1920"
                    xmlns="http://www.w3.org/2000/svg"
                    ><g id="SVGRepo_bgCarrier" stroke-width="0" /><g
                        id="SVGRepo_tracerCarrier"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    /><g id="SVGRepo_iconCarrier">
                        <path
                            d="M735.385 336.094c218.24 0 395.977 177.624 395.977 395.976v113.137c0 121.96-56.568 229.78-143.57 302.526 94.13 13.916 187.354 34.959 278.315 64.6 122.414 39.825 204.664 155.676 204.664 288.159v200.364l-26.814 16.63c-148.434 92.32-392.017 202.515-708.572 202.515-174.795 0-439.76-35.186-708.685-202.514L0 1700.856v-189.39c0-140.629 89.264-263.042 221.973-304.79 85.418-26.7 172.533-46.498 260.327-59.509-86.55-72.746-142.891-180.339-142.891-301.96V732.07c0-218.352 177.623-395.976 395.976-395.976ZM1183.405 0c218.24 0 395.976 177.624 395.976 395.977v113.136c0 121.96-56.568 229.893-143.57 302.526 94.13 13.916 187.241 35.072 278.316 64.6 122.413 40.051 204.663 155.79 204.663 288.272v200.364l-26.7 16.631c-77.612 48.31-181.81 101.03-308.183 140.742v-21.723c0-181.696-113.589-340.766-282.727-395.75a1720.133 1720.133 0 0 0-111.553-32.244c35.751-69.805 54.871-147.416 54.871-227.29V732.104c0-250.483-182.036-457.975-420.414-500.175C886.762 95.487 1023.656 0 1183.404 0Z"
                            fill-rule="evenodd"
                        />
                    </g></svg
                >
            </span>
            <span
                class="img-overlay"
                id="group_icon_btn"
                style="display: block;margin-left: 18%;"
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
                >
                    <path
                        fill="currentColor"
                        d="M21.317,4.381H10.971L9.078,2.45C8.832,2.199,8.342,1.993,7.989,1.993H4.905 c-0.352,0-0.837,0.211-1.078,0.468L1.201,5.272C0.96,5.529,0.763,6.028,0.763,6.38v1.878c0,0.003-0.002,0.007-0.002,0.01v11.189 c0,1.061,0.86,1.921,1.921,1.921h18.634c1.061,0,1.921-0.86,1.921-1.921V6.302C23.238,5.241,22.378,4.381,21.317,4.381z  M12.076,18.51c-3.08,0-5.577-2.497-5.577-5.577s2.497-5.577,5.577-5.577s5.577,2.497,5.577,5.577 C17.654,16.013,15.157,18.51,12.076,18.51z M12.076,9.004c-2.17,0-3.929,1.759-3.929,3.929s1.759,3.929,3.929,3.929 s3.929-1.759,3.929-3.929C16.004,10.763,14.245,9.004,12.076,9.004z"
                    />
                </svg>
                <br />
                <span>Add a group icon</span>
            </span>
            <input
                type="file"
                id="group_file_input"
                bind:files={uploadedFile}
                hidden
            />
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

    .search-bar {
        width: 85%;
        padding: 0;
        margin-left: 10px;
        border-radius: 15px;
        display: inline-block;
        margin-bottom: 10px;
        background-color: #202c33;
    }
    .search-bar svg {
        color: #f0ffff80;
        font-size: 7px;
        margin: 3px;
        position: absolute;
    }
    .chat-search {
        width: 86%;
        margin: 0;
        margin-left: 27px;
        background-color: #202c33;
        border: none;
        color: #f0ffff;
        outline: none;
    }

    .filter {
        color: #f0ffff80;
        cursor: pointer;
    }

    #filter {
        position: absolute;
        border-radius: 50%;
        padding: 2px;
        width: 25px;
        height: 25px;
        margin-left: 10px;
        text-align: center;
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
        background-color: rgba(90, 90, 90, 0.8);
        padding: 10px;
        width: 180px;
        height: 180px;
        margin-left: 110px;
        margin-top: -200px;
        color: azure;
        display: none;
        cursor: pointer;
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

    .user-search {
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

    .new-group {
        width: 100%;
        height: 100%;
        padding: 10px 0;
        color: azure;
    }

    .group-members {
        display: flex;
        flex-wrap: wrap;
    }

    .new-group ul {
        margin: 0;
        padding: 0;
    }

    .new-group li {
        margin: 0;
        padding: 5px;
        cursor: pointer;
        display: flex;
        flex-wrap: wrap;
        list-style-type: none;
    }

    .new-group li img {
        border-radius: 50%;
        width: 50px;
        height: 50px;
        margin: 5px;
    }

    .new-group li .name {
        text-align: center;
        padding: 2px;
        margin-top: 12px;
        left: 75px;
        position: absolute;
    }

    .new-group li .bio-text {
        left: 75px;
        font-size: 12px;
        margin-top: 35px;
        text-align: center;
        position: absolute;
    }

    .new-group li:hover {
        background-color: #202c33;
    }
    .new-member {
        display: flex;
        flex-wrap: wrap;
        margin: 5px;
    }
    .new-member img {
        border-radius: 50%;
        width: 20px;
        height: 20px;
        margin: 2px;
    }

    .new-member .member-name {
        font-size: 13px;
        margin-top: 4px;
    }

    .member-cancel {
        cursor: pointer;
        margin-left: 5px;
        font-size: 19px;
    }

    .group-btn {
        color: azure;
        border-radius: 50%;
        width: 46px;
        height: 46px;
        text-align: center;
        position: fixed;
        bottom: 20px;
        left: 10%;
        background-color: #00a884;
    }
    .group-btn svg {
        margin-top: 8px;
    }

    .side-topbar h3 {
        top: 0px;
        left: 70px;
        color: #f0ffff;
        position: absolute;
    }
    .group-bg {
        display: block;
        text-align: center;
        width: 200px;
        height: 200px;
        border-radius: 50%;
        background-color: #a5dce0;
        margin-left: 18%;
    }
    .group-bg svg {
        margin-top: 35px;
        width: 100px;
        height: 100px;
    }
</style>
