<script>
  import MyModal from "../../components/MyModal.svelte";
  import Swal from "sweetalert2";
  import axios from "axios";
  import config from "../../config";

  let name = "";
  let gender = "male";
  let genderList = [
    { key: "male", value: "ชาย" },
    { key: "female", value: "หญิง" },
  ];
  let phone = "";
  let registerDate = new Date();
  let expireDate = new Date();

  const save = async () => {
    try {
      const payload = {
        name: name,
        phone: phone,
        gender: gender,
        registerDate: registerDate,
        expireDate: expireDate,
      };

      const res = await axios.post(
        config.apiPath + "/api/member/create",
        payload
      );

      if (res.data.message === "success") {
        Swal.fire({
          title: "save",
          text: "saved",
          icon: "success",
        });
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

<div class="card">
  <div class="card-header">สมาชิกร้าน</div>
  <div class="card-body">
    <button
      class="btn btn-primary"
      data-bs-toggle="modal"
      data-bs-target="#modalMember"
    >
      <i class="fa fa-plus mr-2"></i>เพิ่มรายการ
    </button>
  </div>
</div>

<MyModal id="modalMember" title="สมาชิกร้าน">
  <div>
    <div>ชื่อสมาชิก</div>
    <input class="form-control" bind:value={name} />
  </div>
  <div class="mt-3">
    <div>เพศ</div>
    <select class="form-control" bind:value={gender}>
      {#each genderList as item}
        <option value={item.key}>
          {item.value}
        </option>
      {/each}
    </select>
  </div>
  <div class="mt-3">
    <div>เบอร์โทร</div>
    <input class="form-control" bind:value={phone} />
  </div>
  <div class="mt-3">
    <div>วันลงทะเบียน</div>
    <input class="form-control" type="date" bind:value={registerDate} />
  </div>
  <div class="mt-3">
    <div>วันสิ้นสุดการเป็นสมาชิก</div>
    <input class="form-control" type="date" bind:value={expireDate} />
  </div>
  <button class="mt-3 btn btn-primary" on:click={() => save()}>
    <i class="fa fa-check me-2"></i>Save
  </button>
</MyModal>
