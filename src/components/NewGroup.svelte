<script>
  import axios from "axios";
  import { createEventDispatcher } from "svelte";
  import { onMount } from "svelte";

  export let user_data;
  export let members;

  const host = process.env.API_URL;
  const dispatch = createEventDispatcher();

  async function uploadGroupIcon(group) {
    let fileInput = document.getElementById("group_file_input");
    const file = fileInput.files[0];
    const formData = new FormData();
    formData.append("file", file);

    await axios
      .post(
        `${host}/monchat/group_upload/${group.group_id}/${user_data.user_id}/`,
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

  async function createNewGroup() {
    let payload = {
      name: document.getElementById("group_name").value,
      description: document.getElementById("group_desc").value,
      members: members.map((member) => member.user_id),
    };
    console.log(payload);
    await axios
      .post(`${host}/monchat/group/${user_data.user_id}/`, payload, {
        headers: {
          "Content-Type": "application/json",
        },
      })
      .then((response) => {
        if (groupIcon.length > 0) {
          uploadGroupIcon(response.data.data);
        }
        dispatch("back", {
          sidebar: "chats",
        });
        dispatch("groupcreate", { ...response.data.data });
      });
  }

  let groupIcon = [];

  var nameInserted;

  onMount((e) => {
    setTimeout((f) => {
      let fileIn = document.getElementById("group_file_input");
      let btn = document.getElementById("group_icon_btn");
      btn.addEventListener("click", (e) => {
        fileIn.click();
      });
    }, 2000);
  });
</script>

<div class="new_group_main">
  <div class="side-topbar">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span
      class="back-arrow"
      on:click={(e) => {
        dispatch("back", {
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
  <span class="img-overlay" id="group_icon_btn">
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
  <input type="file" id="group_file_input" bind:files={groupIcon} hidden />

  <input
    type="text"
    class="group-name"
    id="group_name"
    placeholder="Group's name"
    bind:value={nameInserted}
  />
  <input
    type="text"
    class="group-desc"
    id="group_desc"
    placeholder="Group's description"
  />

  {#if nameInserted}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span
      class="group-btn"
      on:click={(e) => {
        createNewGroup();
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
          d="M8,17.1l-5.2-5.2l-1.7,1.7l6.9,7L22.9,5.7L21.2,4L8,17.1z"
        /></svg
      >
    </span>
  {/if}
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
  .back-arrow svg {
    color: azure;
    cursor: pointer;
    margin: 12px;
  }

  .group-bg {
    display: block;
    text-align: center;
    width: 200px;
    height: 200px;
    border-radius: 50%;
    background-color: #a5dce0;
    margin-left: 22%;
  }
  .group-bg svg {
    margin-top: 35px;
    width: 100px;
    height: 100px;
  }

  input {
    margin-left: 26px;
    background-color: #202c338c;
    border: none;
    outline: none;
    color: azure;
    border-radius: 14px;
    text-align: center;
    margin-top: 20px;
    width: 85%;
  }
  .group-btn {
    color: azure;
    border-radius: 50%;
    width: 46px;
    height: 46px;
    text-align: center;
    position: fixed;
    bottom: 20px;
    left: 12%;
    background-color: #00a884;
  }
  .group-btn svg {
    margin-top: 8px;
  }

  .img-overlay {
    border-radius: 50%;
    background-color: rgba(90, 90, 90, 0.8);
    padding: 10px;
    width: 180px;
    height: 180px;
    margin-left: 22%;
    margin-top: -200px;
    color: azure;
    display: block;
    cursor: pointer;
    text-align: center;
    position: absolute;
  }

  .img-overlay svg {
    width: 40px;
    height: 40px;
    margin-top: 30px;
  }
  .img-overlay span {
    width: 60px;
  }
</style>
