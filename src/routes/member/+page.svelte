<script>
  import MyModal from "../../components/MyModal.svelte";
  import Swal from "sweetalert2";
  import axios from "axios";
  import config from "../../config";
  import { onMount } from "svelte";
  import dayjs from "dayjs";
  import { expoIn } from "svelte/easing";

  let name = "";
  let gender = "male";
  let genderList = [
    { key: "male", value: "ชาย" },
    { key: "female", value: "หญิง" },
  ];
  let phone = "";
  let registerDate = new Date();
  let expireDate = new Date();
  let members = []; // ไว้เก็บค่าจาก Database มา loop แสดงผล
  let id = 0; // เอาไว้เก็บตอนแก้ไขข้อมูล

  const save = async () => {
    try {
      const payload = {
        name: name,
        phone: phone,
        gender: gender,
        registerDate: registerDate,
        expireDate: expireDate,
      };

      let message = "";

      if (id > 0) {
        const res = await axios.put(
          config.apiPath + "/api/member/update/" + id,
          payload
        );
        message = res.data.message;
      } else {
        const res = await axios.post(
          config.apiPath + "/api/member/create",
          payload
        );
        message = res.data.message;
      }

      if (message === "success") {
        id = 0;

        Swal.fire({
          title: "save",
          text: "saved",
          icon: "success",
          timer: 1000,
        });
        fetchData();
        document.getElementById("modalMember_btnClose").click();
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  onMount(() => {
    fetchData();
  });

  const fetchData = async () => {
    try {
      const res = await axios.get(config.apiPath + "/api/member/list");

      if (res.data.results !== undefined) {
        members = res.data.results;
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const displayGender = (gender) => {
    for (let i = 0; i < genderList.length; i++) {
      if (genderList[i].key == gender) {
        return genderList[i].value;
      }
    }
  };

  const remove = async (item) => {
    try {
      const button = await Swal.fire({
        title: "ลบข้อมูล",
        text: "ยืนยันการลบ",
        icon: "question",
        showCancelButton: true,
        showConfirmButton: true,
      });

      if (button.isConfirmed) {
        await axios.delete(config.apiPath + "/api/member/remove/" + item.id);
        fetchData();
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const edit = (item) => {
    name = item.name;
    phone = item.phone;
    gender = item.gender;
    registerDate = dayjs(item.registerDate).format("YYYY-MM-DD");
    expireDate = dayjs(item.expireDate).format("YYYY-MM-DD");
    id = item.id;
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

    <table class="mt-3 table table-bordered table-striped">
      <thead>
        <tr>
          <th>ชื่อสมาชิก</th>
          <th>เพศ</th>
          <th>เบอร์โทร</th>
          <th>วันลงทะเบียน</th>
          <th>วันสิ้นสุดการเป็นสมาชิก</th>
          <th width="200px"></th>
        </tr>
      </thead>
      <tbody>
        {#each members as item}
          <tr>
            <td>{item.name}</td>
            <td>{displayGender(item.gender)}</td>
            <td>{item.phone}</td>
            <td>{dayjs(item.registerDate).format("DD/MM/YYYY")}</td>
            <td>{dayjs(item.expireDate).format("DD/MM/YYYY")}</td>
            <td class="text-center">
              <button
                class="btn btn-primary"
                data-bs-toggle="modal"
                data-bs-target="#modalMember"
                on:click={() => edit(item)}
              >
                <i class="fa fa-pencil"></i>
              </button>
              <button class="btn btn-danger" on:click={() => remove(item)}>
                <i class="fa fa-times"></i>
              </button>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
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
