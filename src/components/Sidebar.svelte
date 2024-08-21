<script>
  import axios from "axios";
  import { onMount } from "svelte";
  import Swal from "sweetalert2";
  import config from "../config";
  import { goto } from "$app/navigation";

  let name = "";

  onMount(() => {
    fetchData();
  });

  const fetchData = async () => {
    try {
      const token = localStorage.getItem("fitness_token");
      const res = await axios.get(config.apiPath + "/api/user/info", {
        headers: {
          Authorization: token,
        },
      });

      if (res.data.name !== null) {
        name = res.data.name;
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const singOut = async () => {
    const button = await Swal.fire({
      title: "ออกจากระบบ",
      text: "คุณต้องการออกจากระบบ ใช่หรือไม่",
      icon: "question",
      showCancelButton: true,
      showConfirmButton: true,
    });

    if (button.isConfirmed) {
      localStorage.removeItem("fitness_token");
      name = "";
      goto("/");
    }
  };
</script>

<div class="wrapper">
  <aside class="main-sidebar sidebar-dark-primary elevation-4">
    <a href="index3.html" class="brand-link">
      <span class="brand-text font-weight-light">Fitness BackOffice</span>
    </a>

    <div class="sidebar">
      <div class="user-panel mt-3 pb-3 mb-3 d-flex">
        <div class="info">
          {#if name != undefined}
            <div class="d-block text-white">
              User: {name}
              <div class="mt-2">
                <button class="btn btn-danger" on:click={() => singOut()}>
                  <i class="fa fa-times mr-2"></i>Sign Out
                </button>
              </div>
            </div>
          {/if}
        </div>
      </div>

      <nav class="mt-2">
        <ul
          class="nav nav-pills nav-sidebar flex-column"
          data-widget="treeview"
          role="menu"
          data-accordion="false"
        >
          <li class="nav-item">
            <a href="/home" class="nav-link">
              <i class="nav-icon fas fa-th"></i>
              <p>Dashboard</p>
            </a>
          </li>
          <li class="nav-item">
            <a href="/member" class="nav-link">
              <i class="nav-icon fas fa-th"></i>
              <p>สมาชิกร้าน</p>
            </a>
          </li>
          <li class="nav-item">
            <a href="/checkin" class="nav-link">
              <i class="nav-icon fas fa-th"></i>
              <p>บันทึกการเข้ายิม</p>
            </a>
          </li>
          <li class="nav-item">
            <a href="/user" class="nav-link">
              <i class="nav-icon fas fa-th"></i>
              <p>เทรนเนอร์ และพนักงาน</p>
            </a>
          </li>
          <li class="nav-item">
            <a href="/device" class="nav-link">
              <i class="nav-icon fas fa-th"></i>
              <p>อุปกรณ์</p>
            </a>
          </li>
          <li class="nav-item">
            <a href="/course" class="nav-link">
              <i class="nav-icon fas fa-th"></i>
              <p>คอร์สเรียน</p>
            </a>
          </li>
        </ul>
      </nav>
    </div>
  </aside>
</div>
