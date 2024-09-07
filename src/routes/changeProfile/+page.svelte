<script>
  import axios from "axios";
  import Swal from "sweetalert2";
  import config from "../../config";

  let username = "";
  let password = "";

  const save = async () => {
    try {
      const payload = {
        username: username,
        password: password,
      };
      const res = await axios.post(
        config.apiPath + "/api/user/changeProfile",
        payload,
        {
          headers: {
            Authorization: localStorage.getItem("fitness_token"),
          },
        }
      );

      if (res.data.message === "success") {
        location.reload();
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };
</script>

<div class="card mt-3">
  <div class="card-header">เปลี่ยนข้อมูลตัวเอง</div>
  <div class="card-body">
    <div class="alert alert-danger">
      * หากไม่ต้องการเปลี่ยนข้อมูล โปรดอย่ากรอกข้อมูลใดๆ
    </div>
    <div>
      <div>Username</div>
      <input class="form-control" bind:value={username} />
    </div>
    <div class="mt-3">
      <div>Password</div>
      <input class="form-control" type="password" bind:value={password} />
    </div>
    <div class="mt-3">
      <button class="btn btn-primary" on:click={() => save()}>
        <i class="fa fa-check mr-2"></i>Save
      </button>
    </div>
  </div>
</div>
