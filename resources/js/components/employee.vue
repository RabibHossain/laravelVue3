<template>
    <div class="container p-5">

        <button type="button" class="btn btn-primary mb-4 float-right"
        data-toggle="modal" data-target="#myModal" @click="resetForm">
            New Employee
        </button>

        <div class="modal" id="myModal">
            <div class="modal-dialog">
                <div class="modal-content">

                    <div class="modal-header">
                        <h5 class="modal-title text-muted">Add New Employee</h5>
                        <button type="button" class="close close_modal" data-dismiss="modal">&times;</button>
                    </div>

                    <div class="modal-body">

                        <div class="text-sm text-danger" v-if="errors !== '' ">
                            <p v-for="error in errors" :key="error"><small>{{ error }}</small></p>
                        </div>

                        <div class="form-group">
                            <label>Name</label>
                            <input type="text" class="form-control" v-model="form.name">
                        </div>

                        <div class="form-group">
                            <label>Email</label>
                            <input type="text" class="form-control" v-model="form.email">
                        </div>

                        <div class="form-group">
                            <label>Phone</label>
                            <input type="text" class="form-control"  v-model="form.phone">
                        </div>

                        <button v-if="employee_id == ''" type="button" class="btn btn-primary"
                                @click="storeEmployee">Submit</button>

                        <button v-else type="button" class="btn btn-primary"
                                @click="updateEmployee">Update</button>

                    </div>


                </div>
            </div>
        </div>


        <table class="table table-bordered table-hover">
            <thead>
                <th>Name</th>
                <th>Email</th>
                <th>Phone</th>
                <th>Action</th>
            </thead>

            <tbody>
                <template v-for="employee in employees" :key="employee.id">
                    <tr>
                        <td>{{ employee.name }}</td>
                        <td>{{ employee.email }}</td>
                        <td>{{ employee.phone }}</td>
                        <td>
                            <button type="button" class="btn btn-sm btn-info mr-2"
                                    data-toggle="modal" data-target="#myModal" @click="editEmployee(employee)">
                                Edit
                            </button>

                            <button type="button" class="btn btn-sm btn-info" @click="deleteEmployee(employee.id)">
                                Delete
                            </button>
                        </td>
                    </tr>
                </template>

            </tbody>
        </table>

    </div>
</template>

<script>

    import { ref, reactive, onMounted } from "vue";
    import axios from "axios";

    export default {
        setup() {
            const api = "http://localhost:8000/";
            const employees = ref([]);
            const employee_id = ref([]);
            const errors = ref([]);

            const form = reactive({
                name: '',
                email: '',
                phone: ''
            });

            const getEmployees = async() => {
                let res = await axios.get(api +'api/employees');
                employees.value = res.data;
            }

            const storeEmployee = async() => {
                try {
                    await axios.post(api+'api/employee', form)
                    getEmployees()
                    resetForm()
                    $("#myModal").modal('hide');
                } catch(e) {
                    console.log(e);
                    if (e.response.status === 422) {
                        const data = [];
                        for (const key in e.response.data.errors) {
                            data.push(e.response.data.errors[key][0]);
                        }
                        errors.value = data;
                    }
                }
            }

            const updateEmployee = async() => {
                try {
                    await axios.patch(api+'api/employee/'+employee_id.value, form)
                    getEmployees()
                    resetForm()
                    $("#myModal").modal('hide');
                } catch(e) {
                    console.log(e);
                    if (e.response.status === 422) {
                        const data = [];
                        for (const key in e.response.data.errors) {
                            data.push(e.response.data.errors[key][0]);
                        }
                        errors.value = data;
                    }
                }
            }

            const editEmployee = (employee) => {
                employee_id.value = employee.id;
                form.name = employee.name;
                form.email = employee.email;
                form.phone = employee.phone;
            }

            const deleteEmployee = async(id) => {
                await axios.delete(api + 'api/employee/' + id);
                getEmployees();
            }

            onMounted(getEmployees());

            const resetForm = () => {
                employee_id.value = '';
                form.name = '';
                form.email = '';
                form.phone = '';
            }

            return {
                employees,
                form,
                storeEmployee,
                errors,
                editEmployee,
                updateEmployee,
                employee_id,
                resetForm,
                deleteEmployee
            }

        }
    }

</script>
