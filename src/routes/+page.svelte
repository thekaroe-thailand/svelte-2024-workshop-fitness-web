<script>
  import { goto } from "$app/navigation";
  import axios from "axios";
  import Swal from "sweetalert2";
  import config from "../config";

  let username = "";
  let password = "";

  async function signin() {
    try {
      if (username == "" || password == "") {
        Swal.fire({
          title: "ตรวจสอบข้อมูล",
          text: "โปรดกรอกข้อมูลด้วย",
          icon: "warning",
        });
      } else {
        const payload = {
          username: username,
          password: password,
        };

        const res = await axios.post(
          config.apiPath + "/api/user/signIn",
          payload
        );

        if (res.data.token !== undefined) {
          localStorage.setItem("fitness_token", res.data.token);
          goto("/home");
        }
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  }
</script>

<div class="card mt-3">
  <div class="card-header bg-primary">
    Sign In to BackOffice of Fitness Center
  </div>
  <div class="card-body">
    <div>
      <div>Username</div>
      <input class="form-control" bind:value={username} />
    </div>
    <div class="mt-3">
      <div>Password</div>
      <input class="form-control" type="password" bind:value={password} />
    </div>
    <div class="mt-3">
      <button class="btn btn-primary" on:click={() => signin()}>
        <i class="fa fa-check me-2"></i>
        Sign In Now
      </button>
    </div>
  </div>
</div>
