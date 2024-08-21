<script>
  import { onMount } from "svelte";
  import MyModal from "../../components/MyModal.svelte";
  import Swal from "sweetalert2";
  import axios from "axios";
  import config from "../../config";

  let course = {};

  onMount(() => {
    fetchData();
  });

  const fetchData = async () => {};

  const save = async () => {
    try {
      course.price = parseInt(course.price);
      course.dayPerWeek = parseInt(course.dayPerWeek);
      course.hourPerDay = parseInt(course.hourPerDay);

      await axios.post(config.apiPath + "/api/course/create", course);
      fetchData();
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.messsage,
        icon: "error",
      });
    }
  };

  const clearForm = () => {
    course = {};
  };
</script>

<div class="card mt-3">
  <div class="card-header">คอร์สเรียน</div>
  <div class="card-body">
    <button
      class="btn btn-primary"
      data-bs-toggle="modal"
      data-bs-target="#modalCourse"
      on:click={() => clearForm()}
    >
      <i class="fa fa-plus me-2"></i>เพิ่มรายการ
    </button>
  </div>
</div>

<MyModal id="modalCourse" title="คอร์สเรียน">
  <div>
    <div>ชื่อ</div>
    <input class="form-control" bind:value={course.name} />
  </div>
  <div class="mt-3">
    <div>รายละเอียด</div>
    <input class="form-control" bind:value={course.detail} />
  </div>
  <div class="mt-3">
    <div>ราคาต่อเดือน</div>
    <input class="form-control" bind:value={course.price} />
  </div>
  <div class="mt-3">
    <div>วันเรียนต่อสัปดาห์</div>
    <input class="form-control" bind:value={course.dayPerWeek} />
  </div>
  <div class="mt-3">
    <div>ชั่วโมงเรียนต่อวัน</div>
    <input class="form-control" bind:value={course.hourPerDay} />
  </div>
  <div class="mt-3">
    <div>หมายเหตุ</div>
    <input class="form-control" bind:value={course.remark} />
  </div>
  <button class="mt-3 btn btn-primary" on:click={() => save()}>
    <i class="fa fa-check me-2"></i>บันทึก
  </button>
</MyModal>
