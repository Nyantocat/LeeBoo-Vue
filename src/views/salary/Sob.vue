<template>
    <div>
        <!-- 顶部 -->
        <el-input placeholder="请输入关键字" v-model="search" clearable style="width: 500px; text-align: center" size="medium">
            <i slot="prefix" class="el-input__icon el-icon-search"></i>
        </el-input>
        <!-- 添加工资账套按钮 -->
        <el-button type="primary" @click="openDialog(null)" style="margin-left: 20px" size="medium" icon="el-icon-plus" :loading="addLoading">添加工资账套</el-button>

        <!-- 中间部分 -->
        <!-- 工资账套列表 -->
        <el-table :data="tableData.filter(data => !search 
            || data.name.toLowerCase().includes(search.toLowerCase()))"
        ref="salarySobTable" max-height="660" fit @selection-change="handleSelectionChange"
        v-loading="tableLoading" element-loading-text="加载中" style="margin: 20px 0px">
            <!-- 勾选框 -->
            <el-table-column :resizable="false" fixed header-align="center" align="center" type="selection" width="80"></el-table-column>
            <!-- 序号 -->
            <el-table-column :resizable="false" fixed header-align="center" align="center" type="index" prop="id" label="序号" width="80"></el-table-column>
            <!-- 工资账套名称 -->
            <el-table-column :resizable="false" fixed show-overflow-tooltip header-align="center" align="center" prop="name" label="名称" width="180"></el-table-column>
            <!-- 基础工资 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="basicSalary" label="基础工资" width="120">
                <template slot-scope="scope">
                    {{ scope.row.basicSalary + '元' }}
                </template>
            </el-table-column>
            <!-- 午餐补助 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="lunchSalary" label="午餐补助" width="120">
                <template slot-scope="scope">
                    {{ scope.row.lunchSalary + '元' }}
                </template>
            </el-table-column>
            <!-- 交通补助 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="trafficSalary" label="交通补助" width="120">
                <template slot-scope="scope">
                    {{ scope.row.trafficSalary + '元' }}
                </template>
            </el-table-column>
            <!-- 应发工资 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="allSalary" label="应发工资" width="120">
                <template slot-scope="scope">
                    {{ scope.row.allSalary + '元' }}
                </template>
            </el-table-column>
            <!-- 养老保险 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" label="养老保险">
                <!-- 基数 -->
                <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="pensionBase" label="基数" width="100">
                    <template slot-scope="scope">
                        {{ scope.row.pensionBase + '元' }}
                    </template>
                </el-table-column>
                <!-- 比率 -->
                <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="pensionBase" label="比率" width="100">
                    <template slot-scope="scope">
                        {{ scope.row.pensionPer  + '%' }}
                    </template>
                </el-table-column>
            </el-table-column>
            <!-- 医疗保险 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" label="医疗保险">
                <!-- 基数 -->
                <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="medicalBase" label="基数" width="100">
                    <template slot-scope="scope">
                        {{ scope.row.medicalBase + '元' }}
                    </template>
                </el-table-column>
                <!-- 比率 -->
                <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="medicalBase" label="比率" width="100">
                    <template slot-scope="scope">
                        {{ scope.row.medicalPer + '%' }}
                    </template>
                </el-table-column>
            </el-table-column>
            <!-- 公积金 -->
            <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" label="公积金">
                <!-- 基数 -->
                <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="accumulationFundBase" label="基数" width="100">
                    <template slot-scope="scope">
                        {{ scope.row.accumulationFundBase + '元' }}
                    </template>
                </el-table-column>
                <!-- 比率 -->
                <el-table-column :resizable="false" show-overflow-tooltip header-align="center" align="center" prop="accumulationFundBase" label="比率" width="100">
                    <template slot-scope="scope">
                        {{ scope.row.accumulationFundPer + '%' }}
                    </template>
                </el-table-column>
            </el-table-column>
            <!-- 创建时间 -->
            <el-table-column :resizable="false" sortable header-align="center" align="center" prop="createTime" label="创建时间" width="240"></el-table-column>
            <!-- 操作 -->
            <el-table-column :resizable="false" fixed="right" header-align="center" align="center" label="操作" width="220">
                <template slot-scope="scope">
                    <!-- 编辑按钮 -->
                    <el-button size="mini" style="width: 70px" @click="openDialog(scope.row)">
                    <i class="el-icon-edit"></i> 编辑</el-button>
                    <!-- 删除按钮 -->
                    <el-button size="mini" type="danger" style="width: 70px" @click="deleteSalarySob(scope.row.id)">
                    <i class="fa fa-trash"></i> 删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!-- 添加或编辑工资账套对话框 -->
        <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" width="800px" :close-on-click-modal="false" @closed="resetFormData">
            <!-- 步骤条 -->
            <el-steps :active="activeIndex" align-center finish-status="success" style="margin-bottom: 20px">
                <el-step title="账套名称"></el-step>
                <el-step title="基础工资"></el-step>
                <el-step title="午餐补助"></el-step>
                <el-step title="交通补助"></el-step>
                <el-step title="养老保险"></el-step>
                <el-step title="医疗保险"></el-step>
                <el-step title="公积金"></el-step>
            </el-steps>
            
            <!-- 工资账套表单 -->
            <el-form :model="formData" ref="salarySobForm" :rules="formRules" status-icon>
                <!-- 账套名称 -->
                <el-form-item label="账套名称" prop="name" size="medium" v-show="activeIndex === 0" label-width="80px">
                    <el-input type="input" v-model="formData.name" autocomplete="off" style="width: 300px"></el-input>
                </el-form-item>
                <!-- 基础工资 -->
                <el-form-item label="基础工资" prop="basicSalary" size="medium" v-show="activeIndex === 1" label-width="80px">
                    <el-input-number v-model="formData.basicSalary" :min="0" :step="500" label="基础工资"></el-input-number>
                </el-form-item>
                <!-- 午餐补助 -->
                <el-form-item label="午餐补助" prop="lunchSalary" size="medium" v-show="activeIndex === 2" label-width="80px">
                    <el-input-number v-model="formData.lunchSalary" :min="0" :step="100" label="午餐补助"></el-input-number>
                </el-form-item>
                <!-- 交通补助 -->
                <el-form-item label="交通补助" prop="trafficSalary" size="medium" v-show="activeIndex === 3" label-width="80px">
                    <el-input-number v-model="formData.trafficSalary" :min="0" :step="100" label="交通补助"></el-input-number>
                </el-form-item>
                <!-- 养老保险 -->
                <div v-show="activeIndex === 4" style="display: flex; justify-content: flex-start">
                    <!-- 基数 -->
                    <el-form-item label="养老保险基数" prop="pensionBase" size="medium" label-width="108px">
                        <el-input-number v-model="formData.pensionBase" :min="0" :step="200" label="养老保险基数"></el-input-number>
                    </el-form-item>
                    <!-- 比率 -->
                    <el-form-item label="养老保险比率" prop="pensionPer" size="medium" label-width="108px" style="margin-left: 60px">
                        <el-input-number v-model="formData.pensionPer" :min="0" :max="100" :precision="2" label="养老保险比率"></el-input-number>
                    </el-form-item>
                </div>
                <!-- 医疗保险 -->
                <div v-show="activeIndex === 5" style="display: flex; justify-content: flex-start">
                    <!-- 基数 -->
                    <el-form-item label="医疗保险基数" prop="medicalBase" size="medium" label-width="108px">
                        <el-input-number v-model="formData.medicalBase" :min="0" :step="200" label="医疗保险基数"></el-input-number>
                    </el-form-item>
                    <!-- 比率 -->
                    <el-form-item label="医疗保险比率" prop="medicalPer" size="medium" label-width="108px" style="margin-left: 60px">
                        <el-input-number v-model="formData.medicalPer" :min="0" :max="100" :precision="2" label="医疗保险比率"></el-input-number>
                    </el-form-item>
                </div>
                <!-- 公积金 -->
                <div v-show="activeIndex === 6" style="display: flex; justify-content: flex-start">
                    <!-- 基数 -->
                    <el-form-item label="公积金基数" prop="accumulationFundBase" size="medium" label-width="94px">
                        <el-input-number v-model="formData.accumulationFundBase" :min="0" :step="200" label="公积金基数"></el-input-number>
                    </el-form-item>
                    <!-- 比率 -->
                    <el-form-item label="公积金比率" prop="accumulationFundPer" size="medium" label-width="94px" style="margin-left: 60px">
                        <el-input-number v-model="formData.accumulationFundPer" :min="0" :max="100" :precision="2" label="公积金比率"></el-input-number>
                    </el-form-item>
                </div>
            </el-form>
            
            <!-- 对话框底部按钮 -->
            <span slot="footer">
                <el-button @click="leftButton" size="medium">{{ leftButtonText }}</el-button>
                <el-button type="primary" @click="rightButton" size="medium">{{ rightButtonText }}</el-button>
            </span>
            
        </el-dialog>

        <!-- 底部 -->
        <!-- 批量删除工资账套按钮 -->
        <el-button type="danger" @click="deleteSelection" size="medium" icon="fa fa-trash" :loading="batchDeleteLoading"> 批量删除</el-button>
        <!-- 取消全部的选中项按钮 -->
        <el-button @click="cancelSelection" size="medium" icon="fa fa-ban"> 全部取消</el-button>
    </div>
</template>

<script>
import * as SalarySob from '@/api/salarySob'
import * as Message from '@/utils/message'
import * as Generic from '@/utils/generic'

export default {
    data() {
        return {
            // 批量删除中提示
            batchDeleteLoading: false,
            // 添加中提示
            addLoading: false,
            // 表格加载中提示
            tableLoading: true,
            // 搜索内容
            search: '',
            // 添加或编辑时表单的工资账套
            formData: {
                id: null,
                name: null,
                basicSalary: null,
                lunchSalary: null,
                trafficSalary: null,
                allSalary: null,
                pensionBase: null,
                pensionPer: null,
                medicalBase: null,
                medicalPer: null,
                accumulationFundBase: null,
                accumulationFundPer: null
            },
            // 步骤条激活索引
            activeIndex: 0,
            leftButtonText: '取消',
            rightButtonText: '下一步',
            // 对话框标题
            dialogTitle: '添加工资账套',
            // 校验规则
            formRules: {
                name: [
                    { required: true, message: '名称不能为空！', trigger: 'blur' },
                    { min: 2, max: 12, message: '名称长度必须在2~12之间！',  trigger: 'blur' },
                    { pattern: /^[\w_-（）\u4e00-\u9fa5]{2,12}$/, message: '名称含有非法字符！', trigger: 'blur' }
                ]
            },
            // 是否打开对话框
            dialogVisible: false,
            // 表格数据
            tableData: [],
            // 所有的多选项
            multipleSelection: []
        }
    },
    created() {
        this.refreshAllSalarySob()
    },
    methods: {
        /**
         * 获取所有工资账套
         */
        refreshAllSalarySob(res) {
            // 获取所有工资账套
            SalarySob.getAllSalarySob()
            .then(response => {
                this.tableData = response.data.items
                Message.handle(res)
                this.tableLoading = false
            })
        },
        // 对话框左边按钮的点击事件
        leftButton() {
            // 步骤条变化
            if (this.activeIndex >= 0) {
                this.activeIndex--
            }
            // 按钮变化
            if (this.activeIndex === -1) {
                this.dialogVisible = false
            } else if (this.activeIndex === 0) {
                this.leftButtonText = '取消'
            } else {
                this.leftButtonText = '上一步'
                this.rightButtonText = '下一步'
            }
        },
        // 对话框右边按钮的点击事件
        rightButton() {
            this.$refs.salarySobForm.validate((valid) => {
                if (valid) {
                    // 步骤条变化
                    if (this.activeIndex <= 6) {
                        this.activeIndex++
                    }
                    // 按钮变化
                    if (this.activeIndex === 7) {
                        this.addOrEditSalarySob()
                    } else if (this.activeIndex === 7) {
                        this.rightButtonText = '确定'
                    } else {
                        this.rightButtonText = '下一步'
                        this.leftButtonText = '上一步'
                    }
                }
            })
        },
        /**
         * 点击添加或编辑按钮时打开对话框，对话框会动态适应
         * 如果是修改操作就传入当前要编辑的工资账套，反之添加就传入null
         */
        openDialog(currentSalarySob) {
            // 根据方法是否传入工资账套来判断是添加或是编辑对话框
            if (currentSalarySob) {
                this.dialogTitle = '修改工资账套'

                // 数据回显
                this.$nextTick(() => {
                    this.formData = Generic.deeper(currentSalarySob)
                })
            } else {
                this.dialogTitle = '添加工资账套'
            }
            this.dialogVisible = true
        },
        /**
         * 添加或修改工资账套
         */
        addOrEditSalarySob() {
            // 点击了确定后立马关闭对话框并开启加载中提示，提高用户体验
            this.dialogVisible = false
            this.tableLoading = true

            // 添加或修改前的数据非空校验
            this.formData.basicSalary = this.formData.basicSalary == null ? 0 : this.formData.basicSalary
            this.formData.lunchSalary = this.formData.lunchSalary == null ? 0 : this.formData.lunchSalary
            this.formData.trafficSalary = this.formData.trafficSalary == null ? 0 : this.formData.trafficSalary
            this.formData.pensionBase = this.formData.pensionBase == null ? 0 : this.formData.pensionBase
            this.formData.pensionPer = this.formData.pensionPer == null ? 0 : this.formData.pensionPer
            this.formData.medicalBase = this.formData.medicalBase == null ? 0 : this.formData.medicalBase
            this.formData.medicalPer = this.formData.medicalPer == null ? 0 : this.formData.medicalPer
            this.formData.accumulationFundBase = this.formData.accumulationFundBase == null ? 0 : this.formData.accumulationFundBase
            this.formData.accumulationFundPer = this.formData.accumulationFundPer == null ? 0 : this.formData.accumulationFundPer

            // 如果当前表单数据的id属性有值就代表要修改，反之就表示添加
            if (this.formData.id) {
                // 修改工资账套
                SalarySob.updateSalarySob(this.formData)
                .then(response => {
                    this.refreshAllSalarySob(response)
                })
            } else {
                this.addLoading = true
                // 添加工资账套
                SalarySob.addSalarySob(this.formData)
                .then(response => {
                    this.refreshAllSalarySob(response)
                    this.addLoading = false
                })
            }
        },
        /**
         * 删除工资账套
         */
        deleteSalarySob(id) {
            // 再次确认删除
            this.$confirm('是否确定要删除这条数据?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                // 删除指定工资账套
                this.tableLoading = true
                SalarySob.deleteSalarySobById(id)
                .then(response => {
                    this.refreshAllSalarySob(response)
                })
            }).catch(action => {})
        },
        /**
         * 批量删除选中项
         */
        deleteSelection() {
            // 再次确认删除
            this.$confirm(`是否确定要删除这 ${this.multipleSelection.length} 条数据?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                if (this.multipleSelection.length > 0) {
                    this.batchDeleteLoading = true
                    this.tableLoading = true
                    // 提取出所有选中工资账套的id并封装成一个数组
                    let ids = []
                    this.multipleSelection.forEach(selection => {
                        ids.push(selection.id)
                    })
                    
                    // 批量删除选中的工资账套，删除成功后刷新工资账套列表
                    SalarySob.deleteBatchSalarySobByIds(ids)
                    .then(response => {
                        this.refreshAllSalarySob(response)
                        this.batchDeleteLoading = false
                    })
                }
            }).catch(action => {})
        },
        /**
         * 取消所有选中项
         */
        cancelSelection() {
            if (this.multipleSelection.length > 0) {
                this.$refs.salarySobTable.clearSelection()
            }
        },
        /**
         * 更新选中的多选项
         */
        handleSelectionChange(val) {
            this.multipleSelection = val
        },
        /**
         * 重置表单并清除校验结果
         */
        resetFormData() {
            this.$refs.salarySobForm.resetFields()
            this.activeIndex = 0
            this.leftButtonText = '取消'
            this.rightButtonText = '下一步'
        }
    }
}
</script>