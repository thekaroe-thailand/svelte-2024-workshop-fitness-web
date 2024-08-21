<script>
  import MyModal from "../../components/MyModal.svelte";
  import Swal from "sweetalert2";
  import axios from "axios";
  import config from "../../config";
  import { onMount } from "svelte";
  import { applyAction } from "$app/forms";

  let id = 0;
  let name = "";
  let level = "employee";
  let salary = 0;
  let phone = "";
  let address = "";
  let gender = "male";
  let employees = []; // เก็บข้อมูลพนักงาน
  let filter = {};

  const clearForm = () => {
    name = "";
    gender = "male";
    level = "employee";
    salary = 0;
    phone = "";
    address = "";
  };

  const save = async () => {
    try {
      const payload = {
        name: name,
        gender: gender,
        level: level,
        salary: parseInt(salary),
        phone: phone,
        address: address,
      };

      if (id > 0) {
        await axios.put(
          config.apiPath + "/api/employeeAndTrainer/update/" + id,
          payload
        );

        id = 0;
      } else {
        await axios.post(
          config.apiPath + "/api/employeeAndTrainer/create",
          payload
        );
      }

      fetchData();

      document.getElementById("modalEmployee_btnClose").click();
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const fetchData = async () => {
    try {
      const res = await axios.get(
        config.apiPath + "/api/employeeAndTrainer/list"
      );

      if (res.data.results !== undefined) {
        employees = res.data.results;
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

  const labelGender = (item) => {
    if (item == "male") {
      return "ชาย";
    } else {
      return "หญิง";
    }
  };

  const labelLevel = (item) => {
    if (item == "employee") {
      return "พนักงาน";
    } else {
      return "เทรนเนอร์";
    }
  };

  const labelStatus = (item) => {
    if (item == "use") {
      return "ปกติ";
    } else {
      return "ไม่ได้ทำงาน";
    }
  };

  const remove = async (item) => {
    try {
      const button = await Swal.fire({
        title: "ลบรายการ",
        text: "คุณต้องการลบรายการ ใช่หรือไม่",
        icon: "question",
        showCancelButton: true,
        showConfirmButton: true,
      });

      if (button.isConfirmed) {
        await axios.delete(
          config.apiPath + "/api/employeeAndTrainer/remove/" + item.id
        );
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

  const chooseItem = (item) => {
    name = item.name;
    id = item.id;
    gender = item.gender;
    phone = item.phone;
    address = item.address;
    level = item.level;
    salary = item.salary;
  };

  const handleFilter = async () => {
    try {
      const res = await axios.post(
        config.apiPath + "/api/employeeAndTrainer/filter",
        filter
      );

      if (res.data.results !== undefined) {
        employees = res.data.results;
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
  <div class="card-header">พนักงานและ เทรนเนอร์</div>
  <div class="card-body">
    <button
      class="btn btn-primary"
      data-bs-toggle="modal"
      data-bs-target="#modalEmployee"
      on:click={() => clearForm()}
    >
      <i class="fa fa-plus me-2"></i>เพิ่มรายการ
    </button>

    <div class="alert alert-info mt-2">
      <div class="row">
        <div class="col-3">
          <select class="form-control" bind:value={filter.gender}>
            <option value="all">--- ทุกเพศ ---</option>
            <option value="male">ชาย</option>
            <option value="female">หญิง</option>
          </select>
        </div>
        <div class="col-3">
          <select class="form-control" bind:value={filter.level}>
            <option value="all">--- ทุกระดับ ---</option>
            <option value="employee">พนักงาน</option>
            <option value="trainer">เทรนเนอร์</option>
          </select>
        </div>
        <div class="col-3">
          <select class="form-control" bind:value={filter.status}>
            <option value="all">--- ทุกสถานะ ---</option>
            <option value="use">ปกติ</option>
            <option value="delete">ไม่ได้ทำงาน</option>
          </select>
        </div>
        <div class="col-3">
          <button class="btn btn-primary" on:click={() => handleFilter()}>
            <i class="fa fa-search me-2"></i>แสดงข้อมูล
          </button>
        </div>
      </div>
    </div>

    <table class="table mt-3 table-bordered table-striped">
      <thead>
        <tr>
          <th>ชื่อ</th>
          <th>เพศ</th>
          <th>เบอร์โทร</th>
          <th>ที่อยู่</th>
          <th>ระดับ</th>
          <th class="text-right">เงินเดือน</th>
          <th>สถานะ</th>
          <th width="110px"></th>
        </tr>
      </thead>
      <tbody>
        {#each employees as item}
          <tr>
            <td>{item.name}</td>
            <td>{labelGender(item.gender)}</td>
            <td>{item.phone}</td>
            <td>{item.address}</td>
            <td>{labelLevel(item.level)}</td>
            <td class="text-right">{item.salary.toLocaleString("th-TH")}</td>
            <td>{labelStatus(item.status)}</td>
            <td class="text-center">
              {#if item.status == "use"}
                <button
                  class="btn btn-primary"
                  data-bs-toggle="modal"
                  data-bs-target="#modalEmployee"
                  on:click={() => chooseItem(item)}
                >
                  <i class="fa fa-pencil"></i>
                </button>
                <button class="btn btn-danger" on:click={() => remove(item)}>
                  <i class="fa fa-times"></i>
                </button>
              {/if}
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>

<MyModal id="modalEmployee" title="ข้อมูลพนักงาน และเทรนเนอร์">
  <div>
    <div>ชื่อ</div>
    <input class="form-control" bind:value={name} />
  </div>
  <div class="mt-3">
    <div>เพศ</div>
    <input type="radio" value="male" name="gender" bind:group={gender} /> ผู้ชาย
    <input
      type="radio"
      value="female"
      name="gender"
      class="ms-2"
      bind:group={gender}
    /> ผู้หญิง
  </div>
  <div class="mt-3">
    <div>เบอร์โทร</div>
    <input class="form-control" bind:value={phone} />
  </div>
  <div class="mt-3">
    <div>ที่อยู่</div>
    <input class="form-control" bind:value={address} />
  </div>
  <div class="mt-3">
    <div>เงินเดือน</div>
    <input class="form-control" bind:value={salary} />
  </div>
  <div class="mt-3">
    <div>ระดับ</div>
    <input type="radio" value="employee" name="level" bind:group={level} />
    พนักงาน
    <input type="radio" value="trainer" name="level" bind:group={level} /> เทรนเนอร์
  </div>
  <button class="mt-3 btn btn-primary" on:click={() => save()}>
    <i class="fa fa-check me-2"></i>บันทึก
  </button>
</MyModal>
