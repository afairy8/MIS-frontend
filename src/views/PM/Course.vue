<template>
    <div style="padding: 20px">
        <v-data-table
            :headers="headers"
            :items="offering_courses"
            :search="search"
            class="elevation-1"
        >
            <template v-slot:top>
                <v-toolbar flat color="white">
                    <v-toolbar-title>管理开设课程</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-text-field
                        v-model="search"
                        append-icon="mdi-magnify"
                        label="搜索"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-spacer></v-spacer>
                    <v-dialog v-model="dialog" max-width="500px">
                        <template v-slot:activator="{ on }">
                            <v-btn color="primary" dark class="mb-2" v-on="on">新建</v-btn>
                            <download-excel :data="json_data" :fields="json_fields" :before-generate="startDownload"
                                :name="xls_name" style="padding:0px 5px">
                                <v-btn color="primary" dark class="mb-2">导出全部</v-btn>
                            </download-excel>
                            <v-btn color="primary" dark class="mb-2" @click="open_file()">-导入-</v-btn>
                            <input type="file" id="openfile" style="display:none"/>
                        </template>
                        <v-card>
                            <v-card-title>
                                <span class="headline">{{ formTitle }}</span>
                            </v-card-title>

                            <v-card-text>
                                <v-container>
                                    <v-row>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field
                                                v-model="editedItem.course_property"
                                                label="课程性质"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field
                                                v-model="editedItem.course_type"
                                                label="课程类型"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field v-model="editedItem.name" label="名称"></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field v-model="editedItem.credit" label="学分"></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field
                                                v-model="editedItem.teaching_period"
                                                label="教学学时"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field
                                                v-model="editedItem.total_period"
                                                label="总学时"
                                            ></v-text-field>
                                        </v-col>
                                    </v-row>
                                </v-container>
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="blue darken-1" text @click="close">取消</v-btn>
                                <v-btn color="blue darken-1" text @click="save">保存</v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:item.actions="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
                <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
            </template>
            <template v-slot:no-data>暂无数据</template>
        </v-data-table>
    </div>
</template>

<script>
export default {
    data: () => ({
        search: "",
        course_property_selector: [],
        course_type_selector: [], // TODO
        dialog: false,
        headers: [
            { text: "序号", align: "start", value: "id" },
            { text: "课程性质", value: "course_property" },
            { text: "课程类型", value: "course_type" },
            { text: "名称", value: "name" },
            { text: "学分", value: "credit" },
            { text: "教学学时", value: "teaching_period" },
            { text: "总学时", value: "total_period" },
            { text: "操作", value: "actions", sortable: false }
        ],
        offering_courses: [],
        editedIndex: -1,
        editedItem: {
            course_property: "",
            course_type: "",
            name: "",
            teaching_period: 0
        },
        defaultItem: {
            course_property: "",
            course_type: "",
            name: "",
            teaching_period: 0
        },
        json_fields: {
            "课程编号": "number",
            "课程名": "name",
            "课程类型": "course_type",
            "课程性质": "course_property",
            "学分": "credit",
            "总学时": "total_period",
            "课堂教学学时": "teaching_period",
            "课堂实验学时": "experiment_period",
            "课程实习": "practice_period",
            "开设学期": "semester"
        },
        json_data: [],
        xls_name: "开设课程.xls"
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1 ? "新建" : "修改";
        },
        requirements() {
            return this.$store.state.requirements;
        }
    },

    watch: {
        dialog(val) {
            val || this.close();
        }
    },

    created() {
        this.$axios.get("plan/offering_courses/").then(response => {
            this.offering_courses = response.data.offering_courses;
        });
    },

    methods: {
        editItem(item) {
            console.log(item);
            this.editedIndex = this.offering_courses.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.editedItem.rough_requirement = 1;
            this.dialog = true;
        },

        deleteItem(item) {
            const index = this.offering_courses.indexOf(item);
            if (confirm("确定要删除吗?")) {
                this.$axios
                    .delete("plan/offering_courses/", {
                        id: item.id
                    })
                    .then(response => {
                        console.log(response);
                        alert("Done");
                        this.offering_courses.splice(index, 1);
                    });
            }
        },

        close() {
            this.dialog = false;
            setTimeout(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            }, 300);
        },

        save() {
            if (this.editedIndex > -1) {
                this.$axios
                    .put("plan/offering_courses/", {
                        offering_courses: [this.editedItem]
                    })
                    .then(response => {
                        console.log(response);
                        alert("修改成功");
                        Object.assign(
                            this.offering_courses[this.editedIndex],
                            this.editedItem
                        );
                        this.close();
                    });
            } else {
                this.$axios
                    .post("plan/offering_courses/", {
                        offering_courses: [this.editedItem]
                    })
                    .then(response => {
                        console.log(response);
                        alert("添加成功");
                        this.offering_courses.push(this.editedItem);
                        this.close();
                    });
            }
        },

        async startDownload() {
                console.log('show loading');
                await this.$axios
                    .get('plan/offering_courses/')
                    .then((response) => {
                        let offering_courses = response.data["offering_courses"]
                        //表体
                        this.json_data = []
                        for(let offering_course of offering_courses){
                            this.json_data.push({
                                "number": offering_course.number,
                                "name": offering_course.name,
                                "course_type": offering_course.course_type,
                                "course_property": offering_course.course_property,
                                "credit": offering_course.credit,
                                "total_period": offering_course.total_period,
                                "teaching_period": offering_course.teaching_period,
                                "experiment_period": offering_course.experiment_period,
                                "practice_period": offering_course.practice_period,
                                "semester": offering_course.semester+"'"
                            })
                        }
                    })
        },

        open_file() {
            document.getElementById('openfile').click()
        }
    }
};
</script>

