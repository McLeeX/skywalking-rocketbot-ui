<!-- Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to You under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License. -->

<template>
  <div class="profile-task-list flex-v">
    <div class="profile-task-wrapper flex-v">
      <div class="rk-profile-t-tool flex-h">
        Task List
      </div>
      <div class="rk-trace-t-wrapper scroll_hide">
        <table class="rk-trace-t">
          <tr class="rk-trace-tr cp" v-for="(i, index) in taskListSource" @click="selectTask(i)" :key="index">
            <td
              class="rk-trace-td"
              :class="{
                selected: selectedTask.id === i.id,
              }"
            >
              <div class="ell mb-5">
                <span class="b">{{ i.endpointName }}</span>
                <a class="profile-btn bg-blue r" @click="viewTask(i)">
                  <span class="vm">{{ $t('taskView') }}</span>
                </a>
              </div>
              <div class="grey ell sm">
                <span class="mr-10 sm">{{ i.startTime | dateformat }}</span>
                <span class="mr-10 sm">{{ (i.startTime + i.duration * 60 * 1000) | dateformat }}</span>
              </div>
            </td>
          </tr>
        </table>
      </div>
      <rk-sidebox :width="'50%'" :show.sync="viewDetail" :title="$t('taskInfo')">
        <div class="rk-trace-detail">
          <h5 class="mb-10">{{ $t('task') }}.</h5>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ $t('service') }}:</span>
            <span class="g-sm-8 wba">{{ this.selectedTaskService.label }}</span>
          </div>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ $t('endpoint') }}:</span>
            <span class="g-sm-8 wba">{{ this.selectedTask.endpointName }}</span>
          </div>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ this.$t('monitorTime') }}:</span
            ><span class="g-sm-8 wba">{{ this.selectedTask.startTime | dateformat }}</span>
          </div>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ this.$t('monitorDuration') }}:</span
            ><span class="g-sm-8 wba">{{ this.selectedTask.duration }} min</span>
          </div>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ this.$t('minThreshold') }}:</span>
            <span class="g-sm-8 wba">{{ this.selectedTask.minDurationThreshold }} ms</span>
          </div>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ this.$t('dumpPeriod') }}:</span>
            <span class="g-sm-8 wba">{{ this.selectedTask.dumpPeriod }}</span>
          </div>
          <div class="mb-10 clear">
            <span class="g-sm-4 grey">{{ this.$t('maxSamplingCount') }}:</span>
            <span class="g-sm-8 wba">{{ this.selectedTask.maxSamplingCount }}</span>
          </div>
          <h5 class="mb-10" v-if="this.selectedTask.logs" v-show="this.selectedTask.logs.length">{{ $t('logs') }}.</h5>
          <div class="log-item" v-for="(i, index) in this.selectedTask.logs" :key="index">
            <div class="mb-10 sm">
              <span class="mr-10 grey">{{ $t('instance') }}:</span>
              <span>{{ i.instanceName }}</span>
            </div>
            <div class="mb-10 sm">
              <span class="mr-10 grey">{{ $t('operationType') }}:</span>
              <span>{{ i.operationType }}</span>
            </div>
            <div class="mb-10 sm">
              <span class="mr-10 grey">{{ $t('operationTime') }}:</span>
              <span>{{ i.operationTime | dateformat }}</span>
            </div>
          </div>
        </div>
      </rk-sidebox>
    </div>
    <div class="profile-trace-wrapper profile-segment flex-v">
      <div class="rk-profile-t-tool flex-h">
        Sampled Traces
      </div>
      <div class="rk-trace-t-wrapper scroll_hide">
        <table class="rk-trace-t">
          <tr class="rk-trace-tr cp" v-for="(i, index) in segmentList" @click="selectTrace(i)" :key="index">
            <td
              class="rk-trace-td"
              :class="{
                'rk-trace-success': !i.isError,
                'rk-trace-error': i.isError,
                selected: selectedKey == i.segmentId,
              }"
            >
              <div
                class="ell mb-5"
                :class="{
                  blue: !i.isError,
                  red: i.isError,
                }"
              >
                <span class="b">{{ i.endpointNames[0] }}</span>
              </div>
              <div class="grey ell sm">
                <span class="rk-tag mr-10 sm">{{ i.duration }} ms</span>{{ parseInt(i.start) | dateformat }}
              </div>
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
  import { Duration, Option } from '@/types/global';
  import { Component, Prop, Vue } from 'vue-property-decorator';
  import { Action, Mutation } from 'vuex-class';

  @Component
  export default class ProfileTaskList extends Vue {
    @Prop() private taskListSource: any;
    @Prop() private segmentList: any;
    @Prop() private headerSource: any;
    @Action('profileStore/GET_SEGMENT_LIST') private GET_SEGMENT_LIST: any;
    @Mutation('profileStore/SET_CURRENT_SEGMENT') private SET_CURRENT_SEGMENT: any;
    @Action('profileStore/GET_SEGMENT_SPANS') private GET_SEGMENT_SPANS: any;
    private selectedKey: string = '';
    private selectedTask: any = {};
    private viewDetail: boolean = false;
    private selectedTaskService: any = {};

    private selectTask(item: { id: string; serviceId: string }) {
      this.selectedTask = item;
      this.selectedTaskService =
        this.headerSource.serviceSource.filter((service: any) => service.key === item.serviceId)[0] || {};
      this.GET_SEGMENT_LIST({ taskID: item.id });
    }

    private viewTask(item: any) {
      this.viewDetail = true;
      this.selectedTask = item;
    }

    private selectTrace(item: { segmentId: string }) {
      this.SET_CURRENT_SEGMENT(item);
      this.selectedKey = item.segmentId;
      this.GET_SEGMENT_SPANS({ segmentId: item.segmentId });
    }
  }
</script>

<style lang="scss">
  .profile-task-list {
    width: 30%;
    min-height: 150px;
    min-width: 350px;
    height: 100%;
    overflow: auto;
    .profile-task-wrapper {
      height: 100%;
      width: 100%;
    }
    .profile-trace-wrapper {
      height: 100%;
      width: 100%;
    }
    .rk-trace-td {
      padding: 8px 10px;
      border-bottom: 1px solid rgba(0, 0, 0, 0.07);
      &.selected {
        background-color: #ededed;
      }
    }
    .rk-profile-t-tool {
      padding: 10px 10px;
      font-weight: bold;
      border-right: 1px solid rgba(0, 0, 0, 0.07);
      border-bottom: 1px solid rgba(0, 0, 0, 0.07);
    }
    .log-item {
      margin-top: 20px;
    }
    .profile-btn {
      color: #fff;
      padding: 1px 3px;
      border-radius: 2px;
      font-size: 12px;
    }
  }
  .profile-segment {
    border-top: 1px solid rgba(0, 0, 0, 0.07);
  }
</style>
