<script>
  import { onMount } from "svelte";
  import MyModal from "../../components/MyModal.svelte";
  import Swal from "sweetalert2";
  import axios from "axios";
  import config from "../../config";

  let course = {};
  let courses = [];
  let id = 0;

  onMount(() => {
    fetchData();
  });

  const fetchData = async () => {
    try {
      const res = await axios.get(config.apiPath + "/api/course/list");

      if (res.data.results !== undefined) {
        courses = res.data.results;
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.messsage,
        icon: "error",
      });
    }
  };

  const save = async () => {
    try {
      course.price = parseInt(course.price);
      course.dayPerWeek = parseInt(course.dayPerWeek);
      course.hourPerDay = parseInt(course.hourPerDay);

      if (id > 0) {
        await axios.put(config.apiPath + "/api/course/update/" + id, course);
        id = 0;
      } else {
        await axios.post(config.apiPath + "/api/course/create", course);
      }

      fetchData();

      document.getElementById("modalCourse_btnClose").click();
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

  const remove = async (item) => {
    try {
      const button = await Swal.fire({
        title: "ลบคอร์สเรียน",
        text: "ยืนยันการลบรายการ",
        icon: "question",
        showCancelButton: true,
        showConfirmButton: true,
      });

      if (button.isConfirmed) {
        await axios.delete(config.apiPath + "/api/course/remove/" + item.id);
        fetchData();
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.messsage,
        icon: "error",
      });
    }
  };

  const chooseItem = (item) => {
    id = item.id;
    course = item;
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

    <table class="mt-3 table table-bordered table-striped">
      <thead>
        <tr>
          <th>ชื่อ</th>
          <th>รายละเอียด</th>
          <th class="text-end">ราคาต่อเดือน</th>
          <th class="text-end">วันเรียนต่อสัปดาห์</th>
          <th class="text-end">ชั่วโมงเรียนต่อวัน</th>
          <th width="110px"></th>
        </tr>
      </thead>
      <tbody>
        {#each courses as item}
          <tr>
            <td>
              <div>{item.name}</div>
              {#if item.remark != ""}
                <div class="text-danger">* {item.remark}</div>
              {/if}
            </td>
            <td>{item.detail}</td>
            <td class="text-end">{item.price.toLocaleString("th-TH")}</td>
            <td class="text-end">{item.dayPerWeek}</td>
            <td class="text-end">{item.hourPerDay}</td>
            <td class="text-center">
              {#if item.status != "delete"}
                <button
                  class="btn btn-primary"
                  data-bs-toggle="modal"
                  data-bs-target="#modalCourse"
                  on:click={() => chooseItem(item)}
                >
                  <i class="fa fa-pencil"></i>
                </button>
                <button class="btn btn-danger" on:click={() => remove(item)}>
                  <i class="fa fa-times"></i>
                </button>
              {:else}
                <span class="text-danger">ถูกยกเลิก</span>
              {/if}
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
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
