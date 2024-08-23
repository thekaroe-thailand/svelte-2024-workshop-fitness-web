<script>
  import { onMount } from "svelte";

  let courses = [];

  onMount(() => {
    fetchData();
  });

  const fetchData = async () => {};
</script>

<div class="mt-3 card">
  <div class="card-header">เทรนเนอร์ ประจำคอร์ส</div>
  <div class="card-body">
    <table class="table table-bordered table-striped">
      <thead>
        <tr>
          <th>คอร์สเรียน</th>
          <th>รายละเอียด</th>
          <th>ราคาต่อเดือน</th>
          <th>วันเรียนต่อสัปดาห์</th>
          <th>ชั่วโมงเรียนต่อวัน</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
