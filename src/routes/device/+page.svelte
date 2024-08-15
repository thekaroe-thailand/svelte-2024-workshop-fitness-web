<script>
  import axios from "axios";
  import { onMount } from "svelte";
  import Swal from "sweetalert2";
  import config from "../../config";
  import MyModal from "../../components/MyModal.svelte";

  let name = "";
  let price = 0;
  let qty = 0;
  let remark = "";
  let devices = [];
  let sumAllRow = 0;

  onMount(() => {
    fetchData();
  });

  const fetchData = async () => {
    try {
      const res = await axios.get(config.apiPath + "/api/device/list");

      if (res.data.results !== undefined) {
        devices = res.data.results;

        for (let i = 0; i < devices.length; i++) {
          let item = devices[i];
          item.sumPerRow = parseInt(item.qty) * parseInt(item.price);
          sumAllRow += item.sumPerRow;
        }
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
      const payload = {
        name: name,
        price: parseInt(price),
        qty: parseInt(qty),
        remark: remark,
      };
      await axios.post(config.apiPath + "/api/device/create", payload);
      fetchData();

      document.getElementById("modalDevice_btnClose").click();
    } catch (e) {
      Swal.fire({
        title: "error",
        text: e.message,
        icon: "error",
      });
    }
  };

  const clearForm = () => {
    name = "";
    remark = "";
    qty = 0;
    price = 0;
  };
</script>

<div class="card mt-3">
  <div class="card-header">อุปกรณ์</div>
  <div class="card-body">
    <button
      class="btn btn-primary"
      data-bs-toggle="modal"
      data-bs-target="#modalDevice"
      on:click={() => clearForm()}
    >
      <i class="fa fa-plus me-2"></i>เพิ่มรายการ
    </button>

    <table class="table mt-3 table-bordered table-striped">
      <thead>
        <tr>
          <th>ชื่อ</th>
          <th class="text-end" width="150px">ราคา</th>
          <th class="text-end" width="150px">จำนวน</th>
          <th class="text-end" width="150px">ยอดรวม</th>
          <th width="110px"></th>
        </tr>
      </thead>
      <tbody>
        {#each devices as item}
          <tr>
            <td>{item.name}</td>
            <td class="text-end">{item.price.toLocaleString("th-TH")}</td>
            <td class="text-end">{item.qty}</td>
            <td class="text-end">{item.sumPerRow.toLocaleString("th-TH")}</td>
            <td class="text-center">
              <button class="btn btn-primary">
                <i class="fa fa-pencil"></i>
              </button>
              <button class="btn btn-danger">
                <i class="fa fa-times"></i>
              </button>
            </td>
          </tr>
        {/each}
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3">รวม</td>
          <td class="text-end">{sumAllRow.toLocaleString("th-TH")}</td>
          <td class="text-end"></td>
        </tr>
      </tfoot>
    </table>
  </div>
</div>

<MyModal id="modalDevice" title="อุปกรณ์">
  <div>
    <div>ชื่อ</div>
    <input class="form-control" bind:value={name} />
  </div>
  <div class="mt-3">
    <div>หมายเหตุ</div>
    <input class="form-control" bind:value={remark} />
  </div>
  <div class="mt-3">
    <div>ราคา</div>
    <input class="form-control" bind:value={price} />
  </div>
  <div class="mt-3">
    <div>จำนวน</div>
    <input class="form-control" bind:value={qty} />
  </div>
  <div class="mt-3">
    <button class="btn btn-primary" on:click={() => save()}>
      <i class="fa fa-check me-2"></i>บันทึก
    </button>
  </div>
</MyModal>
