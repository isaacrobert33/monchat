<script>
    import axios from "axios";
    import { createEventDispatcher } from "svelte";

    export let user_data;
    export let mutated;

    const host = "http://127.0.0.1:8000";
    const dispatch = createEventDispatcher();

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
</script>

<div class="profile-page">
    <div class="side-topbar">
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <span
            class="back-arrow"
            on:click={(e) => {
                dispatch("onback", {
                    sidebar: "chats",
                });
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
        <input type="file" id="file_input" bind:files={uploadedFile} hidden />
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

<style>
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

    .back-arrow svg {
        color: azure;
        cursor: pointer;
        margin: 12px;
    }
</style>
