<script>
  import axios from "axios";
  import { onMount } from "svelte";
  import Swal from "sweetalert2";
  import config from "../../config";
  import MyModal from "../../components/MyModal.svelte";

  let courses = [];
  let trainers = [];
  let courseId = 0;
  let members = [];

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
        text: e.message,
        icon: "error",
      });
    }
  };

  const chooseCourse = async (item) => {
    try {
      courseId = item.id;

      const res = await axios.post(
        config.apiPath + "/api/employeeAndTrainer/filter",
        {
          status: "use",
          level: "trainer",
          gender: "all",
        }
      );

      if (res.data.results !== undefined) {
        trainers = res.data.results;
      }
    } catch (e) {
      Swal.fire({
        title: "errro",
        text: e.message,
        icon: "error",
      });
    }
  };

  const chooseTrainer = async (item) => {
    try {
      const payload = {
        trainerId: item.id,
        courseId: courseId,
      };

      await axios.post(
        config.apiPath + "/api/courseAndTrainer/create",
        payload
      );

      courseId = 0;
      fetchData();

      document.getElementById("modalTrainer_btnClose").click();
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const openModalMember = async () => {
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
</script>

<div class="mt-3 card">
  <div class="card-header">เทรนเนอร์ ประจำคอร์ส</div>
  <div class="card-body">
    <table class="table table-bordered table-striped">
      <thead>
        <tr>
          <th width="250px">เลือกเทรนเนอร์/ลูกเทรน</th>
          <th>คอร์สเรียน</th>
          <th>รายละเอียด</th>
          <th>ราคาต่อเดือน</th>
          <th>วันเรียนต่อสัปดาห์</th>
          <th>ชั่วโมงเรียนต่อวัน</th>
        </tr>
      </thead>
      <tbody>
        {#each courses as item}
          <tr>
            <td class="text-center">
              <button
                class="btn btn-primary"
                data-bs-toggle="modal"
                data-bs-target="#modalTrainer"
                on:click={() => chooseCourse(item)}
              >
                <i class="fa fa-check me-2"></i>เทรนเนอร์
              </button>
              <button
                class="btn btn-success"
                data-bs-toggle="modal"
                data-bs-target="#modalMember"
                on:click={() => openModalMember()}
              >
                <i class="fa fa-user me-2"></i>ลูกเทรน
              </button>
            </td>
            <td>
              <div>{item.name}</div>
              <div class="text-success">
                ผู้สอน: {item.CourseAndTrainers[0]?.Trainer.name ?? "-"}
              </div>
            </td>
            <td>{item.detail}</td>
            <td>{item.price}</td>
            <td>{item.dayPerWeek}</td>
            <td>{item.hourPerDay}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>

<MyModal id="modalTrainer" title="เลือกเทรนเนอร์ประจำคอร์ส">
  <table class="table table-bordered table-striped">
    <thead>
      <tr>
        <th>เลือก</th>
        <th>ชื่อ</th>
        <th>เพศ</th>
      </tr>
    </thead>
    <tbody>
      {#each trainers as item}
        <tr>
          <td>
            <button
              class="btn btn-primary"
              on:click={() => chooseTrainer(item)}
            >
              <i class="fa fa-check me-2"></i>เลือก
            </button>
          </td>
          <td>{item.name}</td>
          <td>{item.gender}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</MyModal>

<MyModal id="modalMember" title="เลือกลูกเทรน">
  <table class="table table-bordered table-striped">
    <thead>
      <tr>
        <th width="110px">เลือก</th>
        <th>ชื่อ</th>
      </tr>
    </thead>
    <tbody>
      {#each members as item}
        <tr>
          <td>
            <button class="btn btn-primary">
              <i class="fa fa-check me-2"></i>เลือก
            </button>
          </td>
          <td>{item.name}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</MyModal>
