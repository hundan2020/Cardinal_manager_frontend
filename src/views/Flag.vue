<template>
    <div>
        <el-button type="primary" @click="generateFlag">{{$t('flag.generate_flag')}}</el-button>
        <el-button type="primary" @click="exportFlagDialogVisible = true">{{$t('flag.export_flag')}}</el-button>
        <el-table :data="flagList" style="width: 100%" stripe v-loading="flagList === null">
            <el-table-column width="80" prop="ID" label="ID"/>
            <el-table-column width="80" prop="TeamID" :label="$t('flag.team')"/>
            <el-table-column width="80" prop="ChallengeID" :label="$t('flag.challenge')"/>
            <el-table-column width="80" prop="GameBoxID" :label="$t('flag.gamebox')"/>
            <el-table-column width="80" prop="Round" :label="$t('flag.round')"/>
            <el-table-column prop="Flag" :label="$t('flag.flag')"/>
        </el-table>
        <br>
        <el-pagination style="text-align:center" background layout="prev, pager, next" :total="total" :page-size="per"
                       @current-change="(p)=>{page = p; getFlags()}"></el-pagination>

        <!-- Export -->
        <el-dialog :title="$t('flag.export_flag')" :visible.sync="exportFlagDialogVisible">
            <el-form label-width="120px">
                <el-form-item :label="$t('challenge.title')">
                    <el-select v-model="exportChallengeID">
                        <el-option
                                v-for="challenge in challengeList"
                                :key="challenge.ID"
                                :label="challenge.Title"
                                :value="challenge.ID">
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <el-button type="primary" @click="exportFlag">{{$t('general.export')}}</el-button>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Flag",

        data: () => ({
            page: 1,
            per: 15,

            total: 0,
            flagList: null,

            challengeList: [],
            exportFlagDialogVisible: false,
            exportChallengeID: null,

        }),

        mounted() {
            this.getFlags()
            this.getChallenges()
        },

        methods: {
            getFlags() {
                this.utils.GET('/manager/flags?per=' + this.per + '&page=' + this.page).then(res => {
                    this.flagList = res.array
                    this.total = res.total
                }).catch(err => this.$message({message: err, type: 'error'}))
            },

            getChallenges() {
                this.utils.GET("/manager/challenges").then(res => {
                    this.challengeList = res
                }).catch(err => this.$message.error(err))
            },

            generateFlag() {
                this.$confirm(this.$i18n.t('flag.generate_flag_confirm'), this.$i18n.t('flag.confirm'), {
                    confirmButtonText: this.$i18n.t('general.apply'),
                    cancelButtonText: this.$i18n.t('general.cancel'),
                    type: 'warning'
                }).then(() => {
                    this.utils.POST('/manager/flag/generate').then(res => {
                        this.getFlags()
                        this.$message({message: res, type: 'success'})
                    }).catch(err => this.$message({message: err, type: 'error'}))
                })
            },

            exportFlag() {
                this.utils.GET("/manager/flag/export?id=" + this.exportChallengeID).then(res => {
                    let blob = new Blob([JSON.stringify(res)], {type: 'text/plain'})
                    let downloadElement = document.createElement("a")
                    let href = window.URL.createObjectURL(blob)
                    downloadElement.href = href
                    downloadElement.download = name
                    downloadElement.click()
                    window.URL.revokeObjectURL(href)
                }).catch(err => this.$message({message: err, type: 'error'}))
            }
        }
    }
</script>

<style scoped>

</style>