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
  <m-popup :ok-text="$t('Rename')" :nameText="$t('Rename')" @ok="_ok" @close="close" :asyn-loading="true">
    <template slot="content">
      <div class="resource-rename-model">
        <m-list-box-f>
          <template slot="name"><strong>*</strong>{{$t('Name')}}</template>
          <template slot="content">
            <el-input
                    id="inputName"
                    type="input"
                    v-model="name"
                    size="small"
                    :placeholder="$t('Please enter name')">
            </el-input>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name">{{$t('Description')}}</template>
          <template slot="content">
            <el-input
                    id="inputDescription"
                    type="textarea"
                    v-model="description"
                    size="small"
                    :placeholder="$t('Please enter description')">
            </el-input>
          </template>
        </m-list-box-f>
      </div>
    </template>
  </m-popup>
</template>
<script>
  import i18n from '@/module/i18n'
  import store from '@/conf/home/store'
  import mPopup from '@/module/components/popup/popup'
  import mListBoxF from '@/module/components/listBoxF/listBoxF'
  export default {
    name: 'resource-file-rename',
    data () {
      return {
        store,
        description: '',
        name: ''
      }
    },
    props: {
      item: Object
    },
    methods: {
      _ok (fn) {
        this._verification().then(res => {
          if (this.name === this.item.alias) {
            return new Promise((resolve, reject) => {
              this.description === this.item.description ? reject({ msg: '内容未修改' }) : resolve()
            })
          } else {
            return this.store.dispatch('resource/resourceVerifyName', {
              fullName: '/' + this.name,
              type: 'FILE'
            })
          }
        }).then(res => {
          return this.store.dispatch('resource/resourceRename', {
            name: this.name,
            description: this.description,
            id: this.item.id,
            type: 'FILE'
          })
        }).then(res => {
          this.$message.success(res.msg)
          this.$emit('onUpDate', res.data)
          fn()
        }).catch(e => {
          fn()
          this.$message.error(e.msg || '')
        })
      },
      _verification () {
        return new Promise((resolve, reject) => {
          if (!this.name) {
            reject({ // eslint-disable-line
              msg: `${i18n.$t('Please enter resource name')}`
            })
          } else {
            resolve()
          }
        })
      },
      close () {
        this.$emit('close')
      }
    },
    watch: {},
    created () {
      let item = this.item || {}
      if (item) {
        this.name = item.alias
        this.description = item.description
      }
    },
    mounted () {
    },
    components: { mPopup, mListBoxF }
  }
</script>
