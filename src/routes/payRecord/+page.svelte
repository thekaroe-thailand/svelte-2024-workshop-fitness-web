<script>
  import Swal from "sweetalert2";
  import MyModal from "../../components/MyModal.svelte";
  import axios from "axios";
  import config from "../../config";
  import { onMount } from "svelte";
  import dayjs from "dayjs";

  let payRecord = {};
  let payRecords = [];

  const clearForm = () => {
    payRecord = {};
  };

  const save = async () => {
    try {
      payRecord.qty = parseInt(payRecord.qty);
      payRecord.price = parseInt(payRecord.price);
      payRecord.payDate = new Date(payRecord.payDate);
      payRecord.remark = payRecord.remark == undefined ? "" : payRecord.remark;

      if (payRecord.id !== undefined) {
        await axios.put(config.apiPath + "/api/payRecord/update", payRecord);
        payRecord.id = undefined;
      } else {
        await axios.post(config.apiPath + "/api/payRecord/create", payRecord);
      }

      fetchData();

      document.getElementById("modalPayRecord_btnClose").click();
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
      const res = await axios.get(config.apiPath + "/api/payRecord/list");

      if (res.data.results !== undefined) {
        payRecords = res.data.results;
      }
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const remove = async (item) => {
    try {
      const button = await Swal.fire({
        title: "คุณต้องการลบรายการค่าใช้จ่าย",
        text: "โปรดยืนยันการลบ",
        icon: "question",
        showCancelButton: true,
        showConfirmButton: true,
      });

      if (button.isConfirmed) {
        await axios.delete(config.apiPath + "/api/payRecord/remove/" + item.id);
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
</script>

<div class="card mt-3">
  <div class="card-header">บันทึกรายจ่าย</div>
  <div class="card-body">
    <button
      class="btn btn-primary"
      data-bs-toggle="modal"
      data-bs-target="#modalPayRecord"
      on:click={() => clearForm()}
    >
      <i class="fa fa-plus me-2"></i>เพิ่มรายการ
    </button>

    <table class="mt-3 table table-bordered table-striped">
      <thead>
        <tr>
          <th>รายการ</th>
          <th class="text-end">จำนวน</th>
          <th class="text-end">ราคา</th>
          <th class="text-end">ยอดรวม</th>
          <th>วันที่</th>
          <th>หมายเหตุ</th>
          <th width="110px"></th>
        </tr>
      </thead>
      <tbody>
        {#each payRecords as item}
          <tr>
            <td>{item.name}</td>
            <td class="text-end">{item.qty}</td>
            <td class="text-end">{item.price.toLocaleString("th-TH")}</td>
            <td class="text-end">
              {(item.qty * item.price).toLocaleString("th-TH")}
            </td>
            <td>{dayjs(item.payDate).format("DD/MM/YYYY")}</td>
            <td>{item.remark}</td>
            <td class="text-center">
              <button
                class="btn btn-primary"
                data-bs-toggle="modal"
                data-bs-target="#modalPayRecord"
                on:click={() => {
                  payRecord = item;
                  payRecord.payDate = dayjs(new Date(payRecord.payDate)).format(
                    "YYYY-MM-DD"
                  );
                }}
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

<MyModal id="modalPayRecord" title="บันทึกรายจ่าย">
  <div>
    <div>ชื่อรายการ</div>
    <input class="form-control" bind:value={payRecord.name} />
  </div>
  <div class="mt-3">
    <div>จำนวน</div>
    <input class="form-control" bind:value={payRecord.qty} />
  </div>
  <div class="mt-3">
    <div>ราคา</div>
    <input class="form-control" bind:value={payRecord.price} />
  </div>
  <div class="mt-3">
    <div>วันที่จ่าย</div>
    <input class="form-control" type="date" bind:value={payRecord.payDate} />
  </div>
  <div class="mt-3">
    <div>หมายเหตุ</div>
    <input class="form-control" bind:value={payRecord.remark} />
  </div>
  <button class="btn btn-primary" on:click={() => save()}>
    <i class="fa fa-check me-2"></i>บันทึก
  </button>
</MyModal>
