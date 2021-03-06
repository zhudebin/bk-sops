/**
* Tencent is pleased to support the open source community by making 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community
* Edition) available.
* Copyright (C) 2017-2020 THL A29 Limited, a Tencent company. All rights reserved.
* Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
* http://opensource.org/licenses/MIT
* Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
* an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
* specific language governing permissions and limitations under the License.
*/
<template>
    <bk-dialog
        width="600"
        :ext-cls="'common-dialog'"
        :theme="'primary'"
        :mask-close="false"
        :header-position="'left'"
        :title="$t('启动记录')"
        :value="show"
        :draggable="true"
        @value-change="dialogValueChange"
        @confirm="$emit('onClose')"
        @cancel="$emit('onClose')">
        <div class="dialog-content" v-bkloading="{ isLoading: loading, opacity: 1 }">
            <bk-table :data="recordData" :max-height="350" ref="recordTable">
                <bk-table-column type="expand" width="30" align="center">
                    <template slot-scope="props">
                        <div v-if="props.row.ex_data" v-html="transformFailInfo(props.row.ex_data)"></div>
                        <div class="no-error" v-else>{{ $t('无异常信息') }}</div>
                    </template>
                </bk-table-column>
                <bk-table-column :label="$t('序号')" prop="id"></bk-table-column>
                <bk-table-column :width="200" :label="$t('启动时间')" prop="start_at"></bk-table-column>
                <bk-table-column :label="$t('是否启动成功')">
                    <template slot-scope="props">
                        {{ props.row.start_success ? $t('是') : $t('否') }}
                    </template>
                </bk-table-column>
                <bk-table-column :width="80" :label="$t('异常信息')">
                    <template slot-scope="props">
                        <span
                            v-if="!props.row.start_success"
                            class="view-btn"
                            @click="$refs.recordTable.toggleRowExpansion(props.row)">
                            {{ $t('查看') }}
                        </span>
                        <span v-else>--</span>
                    </template>
                </bk-table-column>
                <div class="no-data-matched" slot="empty"><NoData /></div>
            </bk-table>
        </div>
    </bk-dialog>
</template>
<script>
    import { mapActions } from 'vuex'
    import NoData from '@/components/common/base/NoData.vue'
    import { errorHandler } from '@/utils/errorHandler.js'

    export default {
        name: 'BootRecordDialog',
        components: {
            NoData
        },
        props: {
            show: {
                type: Boolean,
                default: false
            },
            id: {
                type: Number,
                default: null
            }
        },
        data () {
            return {
                loading: true,
                isShow: this.show,
                recordData: []
            }
        },
        watch: {
            show (val) {
                this.isShow = val
            }
        },
        methods: {
            ...mapActions('admin/', [
                'periodTaskHistory'
            ]),
            async getRecord () {
                try {
                    this.loading = true
                    const resp = await this.periodTaskHistory({ task_id: this.id })
                    this.recordData = resp.objects
                } catch (e) {
                    errorHandler(e, this)
                } finally {
                    this.loading = false
                }
            },
            transformFailInfo (data) {
                if (!data) {
                    return ''
                }
                if (typeof data === 'string') {
                    const info = data.replace(/\n/g, '<br>')
                    return info
                } else {
                    return data
                }
            },
            dialogValueChange () {
                if (this.show) {
                    this.getRecord()
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
    .dialog-content {
        padding: 30px;
    }
    .information-tips {
        word-break: break-all;
    }
    .no-error {
        text-align: center;
    }
    .view-btn {
        color: #3a84ff;
        cursor: pointer;
    }
</style>
