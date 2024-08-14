<script>
    import { onMount } from "svelte";
    import MyModal from "../../components/MyModal.svelte";
    import Swal from "sweetalert2";
    import axios from "axios";
    import config from "../../config";

    let id;
    let name;
    let members = [];
    let checkins = [];

    onMount(() => {
        fetchData();
        fetchDataCheckIn();
    });

    const fetchDataCheckIn = async () => {
        try {
            const res = await axios.get(config.apiPath + "/api/checkin/list");

            if (res.data.results !== undefined) {
                checkins = res.data.results;
            }
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

    const chooseMember = (item) => {
        name = item.name;
        id = item.id;
        document.getElementById("modalMember_btnClose").click();
    };

    const save = async () => {
        try {
            const payload = {
                member_id: id,
            };
            await axios.post(config.apiPath + "/api/checkin/create", payload);
            fetchDataCheckIn();
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
                title: "ลบรายการ",
                text: "ยืนยันการลบรายการนี้",
                icon: "question",
                showCancelButton: true,
                showConfirmButton: true,
            });

            if (button.isConfirmed) {
                await axios.delete(
                    config.apiPath + "/api/checkin/remove/" + item.id,
                );
                fetchDataCheckIn();
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
    <div class="card-header">
        <div class="card-title">บันทึกการเข้ายิม</div>
    </div>
    <div class="card-body">
        <div class="row">
            <div class="col-5">
                <div>สมาชิกร้าน</div>
                <div class="input-group">
                    <input class="form-control" disabled bind:value={name} />
                    <span class="input-group-append">
                        <button
                            class="btn btn-primary me-2"
                            data-bs-toggle="modal"
                            data-bs-target="#modalMember"
                        >
                            <i class="fa fa-search"></i>
                        </button>
                        <button class="btn btn-success" on:click={() => save()}>
                            <i class="fa fa-check me-2"></i>บันทึก
                        </button>
                    </span>
                </div>
            </div>
        </div>

        <table class="table table-bordered table-striped mt-3">
            <thead>
                <tr>
                    <th>สมาชิกร้าน</th>
                    <th width="250px">วันที่ใช้บริการ</th>
                    <th width="60px"></th>
                </tr>
            </thead>
            <tbody>
                {#each checkins as item}
                    <tr>
                        <td>{item.Member.name}</td>
                        <td>{item.checkin_date}</td>
                        <td class="text-center">
                            <button
                                class="btn btn-danger"
                                on:click={() => remove(item)}
                            >
                                <i class="fa fa-times"></i>
                            </button>
                        </td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>

<MyModal id="modalMember" title="เลือกสมาชิก">
    <table class="table table-bordered table-striped">
        <thead>
            <tr>
                <th>เลือก</th>
                <th>ชื่อ</th>
                <th>เบอร์โทร</th>
                <ht>วันหมดอายุสมาชิก</ht>
            </tr>
        </thead>
        <tbody>
            {#each members as item}
                <tr>
                    <td>
                        <button
                            class="btn btn-primary"
                            on:click={() => chooseMember(item)}
                        >
                            <i class="fa fa-check me-2"></i>เลือก
                        </button>
                    </td>
                    <td>{item.name}</td>
                    <td>{item.phone}</td>
                    <td>{item.expireDate}</td>
                </tr>
            {/each}
        </tbody>
    </table>
</MyModal>
