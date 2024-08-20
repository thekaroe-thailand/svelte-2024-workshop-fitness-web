<script>
  import MyModal from "../../components/MyModal.svelte";
  import Swal from "sweetalert2";
  import axios from "axios";
  import config from "../../config";

  let id = 0;
  let name = "";
  let level = "employee";
  let salary = 0;
  let phone = "";
  let address = "";
  let gender = "male";

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

      await axios.post(
        config.apiPath + "/api/employeeAndTrainer/create",
        payload
      );
      fetchData();
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const fetchData = async () => {};
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
