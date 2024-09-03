<script>
  import Swal from "sweetalert2";
  import dayjs from "dayjs";
  import axios from "axios";
  import config from "../../../config";
  import { ssrImportMetaKey } from "vite/runtime";

  let fromDate = dayjs(new Date()).format("YYYY-MM-DD");
  let toDate = dayjs(new Date()).format("YYYY-MM-DD");
  let incomeMemberShips = [];
  let incomeCourseAndMembers = [];
  let sumIncome = 0;
  let sumPay = 0;
  let totalProfit = 0;
  let paysRecords = [];

  let handleSave = async () => {
    try {
      const payload = {
        fromDate: new Date(fromDate),
        toDate: new Date(toDate),
      };

      const res = await axios.post(
        config.apiPath + "/api/report/income",
        payload
      );

      if (res.data.members !== undefined) {
        incomeMemberShips = res.data.members;
        incomeCourseAndMembers = res.data.courseAndMembers;

        sumIncome = 0;

        for (let i = 0; i < incomeMemberShips.length; i++) {
          let item = incomeMemberShips[i];

          sumIncome += parseInt(item.money);
        }

        for (let i = 0; i < incomeCourseAndMembers.length; i++) {
          let item = incomeCourseAndMembers[i];

          sumIncome += parseInt(item.qty) * parseInt(item.price);
        }
      }

      //
      // pays
      //
      const resPay = await axios.post(
        config.apiPath + "/api/report/payBetween",
        payload
      );

      if (resPay.data.results !== undefined) {
        paysRecords = resPay.data.results;

        sumPay = 0;

        for (let i = 0; i < paysRecords.length; i++) {
          let item = paysRecords[i];

          sumPay += parseInt(item.price) * parseInt(item.qty);
        }
      }

      totalProfit = sumIncome - sumPay;
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
  <div class="card-header">รายงานกำไร ขาดทุน</div>
  <div class="card-body">
    <div class="alert alert-info">
      <div class="row">
        <div class="col-4">
          <div>จากวันที่</div>
          <input class="form-control" type="date" bind:value={fromDate} />
        </div>
        <div class="col-4">
          <div>ถึงวันที่</div>
          <input class="form-control" type="date" bind:value={toDate} />
        </div>
        <div class="col-4">
          <div>&nbsp;</div>
          <div>
            <button class="btn btn-primary" on:click={() => handleSave()}>
              <i class="fa fa-search me-2"></i>แสดงรายการ
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="row mt-3">
      <div class="col-4">
        <div class="card bg-success">
          <div class="card-body">
            <div class="h3 text-end">{sumIncome.toLocaleString("th-TH")}</div>
          </div>
        </div>

        <div class="card">
          <div class="card-header bg-success">
            <i class="fa fa-plus me-2"></i>รายได้
          </div>
          <div class="card-body">
            <strong>ค่าสมาชิก</strong>
            <table class="table table-borderd table-striped">
              <thead>
                <tr>
                  <th>ชื่อ</th>
                  <th class="text-end">ยอดเงิน</th>
                </tr>
              </thead>
              <tbody>
                {#each incomeMemberShips as item}
                  <tr>
                    <td>{item.Member.name}</td>
                    <td class="text-end">{item.money}</td>
                  </tr>
                {/each}
              </tbody>
            </table>

            <div class="mt-3">&nbsp;</div>
            <strong>ค่าคอร์ส</strong>
            <table class="table table-bordered table-striped">
              <thead>
                <tr>
                  <th>คอร์ส</th>
                  <th>ผู้เรียน</th>
                  <td class="text-end">ราคา</td>
                  <td class="text-end">จำนวน</td>
                  <td class="text-end">ยอดรวม</td>
                </tr>
              </thead>
              <tbody>
                {#each incomeCourseAndMembers as item}
                  <tr>
                    <td>{item.Course.name}</td>
                    <td>{item.Member.name}</td>
                    <td class="text-end">
                      {item.price.toLocaleString("th-TH")}
                    </td>
                    <td class="text-end">{item.qty}</td>
                    <td class="text-end">
                      {(item.qty * item.price).toLocaleString("th-TH")}
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div class="col-4">
        <div class="card bg-danger">
          <div class="card-body">
            <div class="h3 text-end">{sumPay.toLocaleString("th-TH")}</div>
          </div>
        </div>

        <div class="card">
          <div class="card-header bg-danger">
            <i class="fa fa-minus me-2"></i>
            รายจ่าย
          </div>
          <div class="card-body">
            <table class="table table-bordered table-striped">
              <thead>
                <tr>
                  <th>รายการ</th>
                  <th class="text-end">ยอดเงิน</th>
                  <th class="text-end">จำนวน</th>
                  <th class="text-end">ยอดรวม</th>
                </tr>
              </thead>
              <tbody>
                {#each paysRecords as item}
                  <tr>
                    <td>{item.name}</td>
                    <td class="text-end">
                      {item.price.toLocaleString("th-TH")}
                    </td>
                    <td class="text-end">{item.qty}</td>
                    <td class="text-end">
                      {(item.qty * item.price).toLocaleString("th-TH")}
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div class="col-4">
        <div class="card">
          <div class="card-body bg-primary">
            <div class="h3 text-end">
              {#if totalProfit > 0}
                กำไร {totalProfit.toLocaleString("th-TH")}
              {:else if totalProfit < 0}
                ขาดทุน {totalProfit.toLocaleString("th-TH")}
              {:else}
                0
              {/if}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
