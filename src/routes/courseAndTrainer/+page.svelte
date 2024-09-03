<script>
  import axios from "axios";
  import { onMount } from "svelte";
  import Swal from "sweetalert2";
  import config from "../../config";
  import MyModal from "../../components/MyModal.svelte";
  import dayjs from "dayjs";

  let courses = [];
  let trainers = [];
  let courseId = 0;
  let members = [];
  let courseAndMember = {};
  let membersInCoruse = [];
  let invoiceUrl = "";

  onMount(() => {
    fetchData();
  });

  const printInvoice = async (invoiceId) => {
    const res = await axios.get(
      config.apiPath + "/api/courseAndMember/printInvoice/" + invoiceId
    );

    if (res.data.fileName !== undefined) {
      invoiceUrl = config.apiPath + "/public/" + res.data.fileName;
    }
  };

  const clearForm = () => {
    courseAndMember = {};
  };

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

  const openModalMember = async (courseId) => {
    try {
      courseAndMember.courseId = courseId;

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

  const save = async () => {
    try {
      courseAndMember.createdDate = new Date(courseAndMember.createdDate);
      courseAndMember.expireDate = new Date(courseAndMember.expireDate);
      courseAndMember.qty = parseInt(courseAndMember.qty);

      await axios.post(
        config.apiPath + "/api/courseAndMember/create",
        courseAndMember
      );

      clearForm();

      document.getElementById("modalCourseAndMember_btnClose").click();
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const chooseMember = (memberId) => {
    courseAndMember.memberId = memberId;
  };

  const openModalMemberInCourse = async (courseId) => {
    try {
      const res = await axios.get(
        config.apiPath + "/api/courseAndMember/list/" + courseId
      );

      if (res.data.results !== undefined) {
        membersInCoruse = res.data.results;
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const removeMemberInCourse = async (item) => {
    try {
      const button = await Swal.fire({
        title: "ลบสมาชิกออกจากคอร์สนี้",
        text: "ยืนยันการลบสมาชิก",
        icon: "question",
        showCancelButton: true,
        showConfirmButton: true,
      });

      if (button.isConfirmed) {
        await axios.delete(
          config.apiPath + "/api/courseAndMember/remove/" + item.id
        );
        openModalMemberInCourse(item.courseId);
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
          <th width="400px">เลือกเทรนเนอร์/ลูกเทรน</th>
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
                on:click={() => openModalMember(item.id)}
              >
                <i class="fa fa-user me-2"></i>เลือกลูกเทรน
              </button>
              <button
                class="btn btn-info"
                data-bs-toggle="modal"
                data-bs-target="#modalMemberInCourse"
                on:click={() => openModalMemberInCourse(item.id)}
              >
                <i class="fa fa-users me-2"></i>ผู้ที่ลงเรียน
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
            <button
              class="btn btn-primary"
              data-bs-toggle="modal"
              data-bs-target="#modalCourseAndMember"
              on:click={() => chooseMember(item.id)}
            >
              <i class="fa fa-check me-2"></i>เลือก
            </button>
          </td>
          <td>{item.name}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</MyModal>

<MyModal id="modalCourseAndMember" title="กรอกข้อมูล">
  <div>
    <div>จำนวน</div>
    <input class="form-control" bind:value={courseAndMember.qty} />
  </div>
  <div class="mt-3">
    <div>วันที่เริ่มต้น</div>
    <input
      class="form-control"
      type="date"
      bind:value={courseAndMember.createdDate}
    />
  </div>
  <div class="mt-3">
    <div>วันที่สิ้นสุด</div>
    <input
      class="form-control"
      type="date"
      bind:value={courseAndMember.expireDate}
    />
  </div>
  <div class="mt-3">
    <div>หมายเหตุ</div>
    <input class="form-control" bind:value={courseAndMember.remark} />
  </div>
  <button class="mt-3 btn btn-primary" on:click={() => save()}>
    <i class="fa fa-check me-2"></i>
    บันทึก
  </button>
</MyModal>

<MyModal id="modalMemberInCourse" title="รายชื่อคนที่ลงเรียนในคอร์สนี้">
  <table class="table table-bordered table-striped">
    <thead>
      <tr>
        <th>ชื่อ</th>
        <th>เบอร์โทร</th>
        <th class="text-end">จำนวนเดือน</th>
        <th>วันที่เริ่มต้น</th>
        <th>วันที่สิ้นสุด</th>
        <th width="100px"></th>
      </tr>
    </thead>
    <tbody>
      {#each membersInCoruse as item}
        <tr>
          <td>{item.Member.name}</td>
          <td>{item.Member.phone}</td>
          <td class="text-end">{item.qty}</td>
          <td>{dayjs(item.createdDate).format("DD/MM/YYYY")}</td>
          <td>{dayjs(item.expireDate).format("DD/MM/YYYY")}</td>
          <td class="text-center">
            <button
              class="btn btn-primary"
              data-bs-toggle="modal"
              data-bs-target="#modalPrintInvoice"
              on:click={() => printInvoice(item.id)}
            >
              <i class="fa fa-print"></i>
            </button>
            <button
              class="btn btn-danger"
              on:click={() => removeMemberInCourse(item)}
            >
              <i class="fa fa-times"></i>
            </button>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</MyModal>

<MyModal id="modalPrintInvoice" title="พิมพ์ใบเสร็จรับเงิน">
  <!-- svelte-ignore a11y-missing-attribute -->
  <iframe src={invoiceUrl} width="100%" height="700px"></iframe>
</MyModal>
