/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
  <m-popup :ok-text="$t('Submit')" :nameText="transferData.type.name + $t('Authorize')" @ok="_ok" @close="close" ref="popup">
    <template slot="content">
      <div class="clearfix transfer-model">
        <div class="select-list-box">
          <div class="tf-header">
            <div class="title">{{transferData.type.name}}{{$t('List')}}</div>
            <div class="count">（{{cacheSourceList.length}}）</div>
          </div>
          <div class="scrollbar tf-content">
            <ul>
              <li v-for="(item,$index) in sourceList" :key="$index" @click="_ckSource(item)">
                <span>{{item.name}}</span>
                <a href="javascript:"></a>
              </li>
            </ul>
          </div>
        </div>
        <div class="select-oper-box">&nbsp;</div>
        <div class="select-list-box">
          <div class="tf-header">
            <div class="title">{{$t('Selected')}}{{transferData.type.name}}</div>
            <div class="count">（{{cacheTargetList.length}}）</div>
          </div>
          <div class="scrollbar tf-content">
            <ul>
              <li v-for="(item,$index) in targetList" :key="$index" @click="_ckTarget(item)"><span>{{item.name}}</span></li>
            </ul>
          </div>
        </div>
      </div>
    </template>
  </m-popup>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import mPopup from '@/module/components/popup/popup'

  export default {
    name: 'transfer',
    data () {
      return {
        sourceList: this.transferData.sourceListPrs,
        targetList: this.transferData.targetListPrs,
        cacheSourceList: this.transferData.sourceListPrs,
        cacheTargetList: this.transferData.targetListPrs,
        searchSourceVal: '',
        searchTargetVal: ''
      }
    },
    props: {
      transferData: Object
    },
    methods: {
      _ok () {
        this.$refs.popup.spinnerLoading = true
        setTimeout(() => {
          this.$refs.popup.spinnerLoading = false
          if (this.transferData.type.name === `${i18n.$t('Managing Users')}`) {
            this.$emit('onUpdate', _.map(this.targetList, v => v.id).join(','))
          } else if (this.transferData.type.name === `${i18n.$t('Project')}`) {
            this.$emit('onUpdateAuthProject', _.map(this.targetList, v => v.id).join(','))
          } else if (this.transferData.type.name === `${i18n.$t('Datasource')}`) {
            this.$emit('onUpdateAuthDataSource', _.map(this.targetList, v => v.id).join(','))
          } else if (this.transferData.type.name === `${i18n.$t('UDF Function')}`) {
            this.$emit('onUpdateAuthUdfFunc', _.map(this.targetList, v => v.id).join(','))
          } else if (this.transferData.type.name === `${i18n.$t('K8s Namespace')}`) {
            this.$emit('onUpdateAuthNamespace', _.map(this.targetList, v => v.id).join(','))
          }
        }, 800)
      },
      _sourceQuery () {
        this.sourceList = this.sourceList.filter(v => v.name.indexOf(this.searchSourceVal) > -1)
      },
      _targetQuery () {
        this.targetList = this.targetList.filter(v => v.name.indexOf(this.searchTargetVal) > -1)
      },
      _ckSource (item) {
        this.targetList = this.cacheTargetList
        this.targetList.unshift(item)
        this.searchTargetVal = ''
        let i1 = _.findIndex(this.sourceList, v => item.id === v.id)
        this.sourceList.splice(i1, 1)
        let i2 = _.findIndex(this.cacheSourceList, v => item.id === v.id)
        if (i2 !== -1) {
          this.cacheSourceList.splice(i2, 1)
        }
      },
      _ckTarget (item) {
        this.sourceList = this.cacheSourceList
        this.sourceList.unshift(item)
        this.searchSourceVal = ''
        let i1 = _.findIndex(this.targetList, v => item.id === v.id)
        this.targetList.splice(i1, 1)
        let i2 = _.findIndex(this.cacheTargetList, v => item.id === v.id)
        if (i2 !== -1) {
          this.cacheTargetList.splice(i2, 1)
        }
      },
      close () {
        if (this.transferData.type.name === `${i18n.$t('Managing Users')}`) {
          this.$emit('close')
        } else if (this.transferData.type.name === `${i18n.$t('Project')}`) {
          this.$emit('closeAuthProject')
        } else if (this.transferData.type.name === `${i18n.$t('Datasource')}`) {
          this.$emit('closeAuthDataSource')
        } else if (this.transferData.type.name === `${i18n.$t('UDF Function')}`) {
          this.$emit('closeAuthUdfFunc')
        }
      }
    },
    mounted () {
    },
    watch: {
      searchSourceVal (val) {
        if (!val) {
          this.sourceList = _.cloneDeep(this.cacheSourceList)
          return
        }
        this._sourceQuery()
      },
      searchTargetVal (val) {
        if (!val) {
          this.targetList = _.cloneDeep(this.cacheTargetList)
          return
        }
        this._targetQuery()
      }
    },
    components: { mPopup }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
  .transfer-model {
    padding: 0 20px;
    .select-list-box {
      width: 220px;
      float: left;
      border: 1px solid #dcdee2;
      border-radius: 3px;
      .tf-header {
        height: 36px;
        line-height: 36px;
        background: #f9fafc;
        position: relative;
        border-bottom: 1px solid #dcdee2;
        margin-bottom: 8px;
        .title {
          position: absolute;
          left: 8px;
          top: 0;
        }
        .count {
          position: absolute;
          right: 8px;
          top: 0;
          font-size: 12px;
        }
      }
      .tf-search {
        background: #fff;
        padding: 8px;
        .fa-search {
          color: #999;
        }
      }
      .tf-content {
        height: 280px;
        ul {
          li {
            height: 28px;
            line-height: 28px;
            cursor: pointer;
            span {
              padding-left: 10px;
              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
              width: 210px;
              display: inline-block;
            }
            &:hover {
              background: #f6faff;
            }
          }
        }
      }
    }
    .select-oper-box {
      width: 20px;
      float: left;
    }
  }
</style>
