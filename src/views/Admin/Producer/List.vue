<template>
    <h2 class="text-center pt-5 pb-3">Nhà sản xuất</h2>
    <div class="px-5">
        <button class="btn-black mb-4">
            <RouterLink to="/admin/producer/create" ><i class="fa-sharp fa-solid fa-plus pr-1"></i>
                Thêm nhà sản xuất
            </RouterLink>
        </button>
       <table class="myTable table-striped  table table-bordered bg-white table-hover">
            <thead class="">
                <tr>
                    <th>STT</th>
                    <th>Tên nhà sản xuất</th>
                    <th>Số điện thoại</th>
                    <th>Email</th>
                    <th style="width: 300px;">Tùy chọn</th>
                </tr>
            </thead>
            <tbody>
                <tr :key="index" v-for="(producer, index) in producers">
                    <td>{{ index + 1 }}</td>
                    <td>{{ producer.name }}</td>
                    <td>{{ producer.phone }}</td> 
                    <td>{{ producer.email }}</td> 
                    <td>
                        <RouterLink class="text-a" :to="{name: 'producer.edit', params: {id: producer.id}}">
                            <button class="btn btn-warning mr-1">
                                <i class="fa-solid fa-pen pr-1"></i>
                                Chỉnh sửa
                            </button>
                        </RouterLink>
                        &nbsp
                        <button class="btn btn-danger"  @click="onDelete(producer.id)"><i class="fa-sharp fa-solid fa-trash pr-1"></i>Xóa</button>
                    </td>
                </tr>
            </tbody>

       </table> 
    </div>
</template>

<script>
import axios from 'axios';
import $ from 'jquery';

export default{
    name: 'ProducerList',
    data(){
        return{
            producers: [],
            admin: [],
        }
    },
    async created(){
        this.getProducers();
        if(localStorage.getItem('tokenAdmin') != null){
            const res = await axios.get('admin',{
            headers:{
                Authorization: 'Bearer ' + localStorage.getItem('tokenAdmin')

            }
		})		
		    this.admin = res.data;            
        }      
    },
    beforeUpdate(){

    },
    methods:{
        getProducers(){
            axios.get('producers').then(res => {
                this.producers = res.data;
                setTimeout(() =>{
                    $('.myTable').DataTable({
                        "language": {
                            "search": "Tìm kiếm:",
                            "searchPlaceholder": "Tìm kiếm",
                            "loadingRecords": "Đang tải...",
                            "zeroRecords": "Không tìm thấy kết quả",
                            "lengthMenu": "Hiển thị _MENU_ bản ghi",
                            "info": "Hiển thị _START_ đến _END_ của _TOTAL_ bản ghi",
                            "paginate": {
                                "first": "Trang đầu",
                                "last": "Trang cuối",
                                "next": "Trang sau",
                                "previous": "Trang trước"
                            }
                        },
                        "lengthMenu": [5, 10, 25, 50],
                        "pageLength": 5
                    });
                }, 100);
            });
        },

        onDelete(producerID) {
            let permissionsArray = [];
            this.admin.roles.forEach(element => {
                permissionsArray.push(element.permissions);
            });
            let hasPermission = false;

            permissionsArray.forEach(permissions => {
                hasPermission = permissions.some(permission => permission.id === 4);
                if (hasPermission) {
                    this.$swal.fire({
                        title: 'Bạn có chắc chắn muốn xóa?',
                        icon: 'warning',
                        showDenyButton: false,
                        showCancelButton: true,
                        confirmButtonColor: '#3085d6',
                        cancelButtonColor: '#d33',
                        confirmButtonText: 'Đồng ý',
                        cancelButtonText: 'Hủy',
                    }).then((result) => {
                        if (result.isConfirmed) {
                            axios.delete(`producers/${producerID}`).then(res => {
                                if (res.data.success) {
                                    this.$swal.fire('Đã xóa thành công!', '', 'success');
                                    this.getProducers();
                                }
                            })
                        }
                    })
                } else {
                    this.$swal.fire({
                        icon: 'error',
                        title: 'Bạn không có quyền xóa!',
                    })
                }
            })

        }
    }
}
</script>