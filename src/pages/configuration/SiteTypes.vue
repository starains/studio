<template>
  <q-page class="flex justify-center q-ma-md">
    <q-pull-to-refresh @refresh="refresh">
      <div style="width: 800px">
        <q-toolbar class="text-primary q-my-lg">
          <q-btn flat round dense icon="location_city" />
          <q-toolbar-title>
            场地类型
          </q-toolbar-title>
          <q-btn flat round dense icon="refresh" @click="refreshList">
            <q-tooltip>刷新</q-tooltip>
          </q-btn>
          <q-btn flat round dense icon="more_vert">
            <q-menu>
              <q-list >
                <q-item clickable v-close-popup @click="add">
                  <q-item-section avatar>
                    <q-icon color="primary" name="location_city" />
                  </q-item-section>
                  <q-item-section>添加场地类型</q-item-section>
                </q-item>
                <q-item clickable v-close-popup @click="impDialog">
                  <q-item-section avatar>
                    <q-icon color="primary" name="cloud_upload" />
                  </q-item-section>
                  <q-item-section>导入场地类型</q-item-section>
                </q-item>
              </q-list>
            </q-menu>
          </q-btn>
        </q-toolbar>
        <q-card class="q-ma-md">
          <q-card-section>
            <q-infinite-scroll @load="loadMore" ref="scroll" >
              <q-list highlight separator>
                <q-item v-for="siteType in siteTypes" :key="siteType.id" >
                  <q-item-section avatar v-if="$q.screen.gt.xs">
                    <q-icon color="primary" name="location_city" />
                  </q-item-section>
                  <q-item-section @click="goto(siteType.id)" class="cursor-pointer">
                    <q-item-label >{{siteType.name}}</q-item-label>
                  </q-item-section>
                  <q-item-section side >
                    <div class="text-grey-8 q-gutter-xs">
                      <q-btn color="secondary" size="12px" flat dense round icon="info" @click="goto(siteType.id)">
                        <q-tooltip>详情</q-tooltip>
                      </q-btn>
                      <q-btn size="12px" flat dense round icon="edit" color="secondary" @click="renameSitetype(siteType)">
                        <q-tooltip>重命名</q-tooltip>
                      </q-btn>
                      <q-btn size="12px" flat dense round icon="cloud_download" color="secondary" @click="exp(siteType)">
                        <q-tooltip>导出</q-tooltip>
                      </q-btn>
                      <q-btn size="12px" flat dense round icon="delete" color="red" @click="del(siteType.id)">
                        <q-tooltip>删除</q-tooltip>
                      </q-btn>
                    </div>
                  </q-item-section>
                </q-item>
              </q-list>
            </q-infinite-scroll>
          </q-card-section>
        </q-card>
      </div>
    </q-pull-to-refresh>
    <q-dialog v-model="importDialog">
      <q-card style="width: 400px; max-width: 80vw;">
        <q-toolbar class="text-primary">
          <q-toolbar-title>
            导入场地类型
          </q-toolbar-title>
          <q-space />
          <q-btn icon="close" flat round dense @click="importDialog = false" />
        </q-toolbar>
        <q-card class="q-ma-md">
          <q-card-section>
            <q-list>
              <q-item>
                <q-file color="primary"  v-model="file" label="场地类型文件" >
                  <template v-slot:prepend>
                    <q-icon name="attach_file" />
                  </template>
                </q-file>
              </q-item>
            </q-list>
          </q-card-section>
          <q-card-actions align="right" class="text-primary">
            <q-btn label="确定" color="primary" @click="imp" />
          </q-card-actions>
        </q-card>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { http } from '../../components/http'

export default {
  name: 'site-types',
  data () {
    return {
      siteTypes: [],
      importDialog: false,
      file: null
    }
  },
  computed: {
  },
  methods: {
    renameSitetype (siteType) {
      this.$q.dialog({
        title: '重命名',
        message: '请输入场地类型名称',
        prompt: {
          type: 'text',
          model: siteType.name
        },
        ok: {
          label: '确定'
        }
      }).onOk((data) => {
        let editUrl = '/sitetypes/name'
        let info = {
          'id': siteType.id,
          'name': data
        }
        http('patch', editUrl, info, (response) => {
          this.refreshList()
        })
      })
    },
    impDialog () {
      this.importDialog = true
    },
    imp () {
      if (this.file !== null) {
        let formData = new FormData()
        formData.append('file', this.file)
        http('post', '/sitetypes/import', formData, (response) => {
          this.refreshList()
        })
        this.importDialog = false
        this.file = null
      } else {
        this.$q.dialog({
          title: '警告',
          message: '请选择场地类型文件。',
          ok: {
            label: '确定'
          }
        })
      }
    },
    exp (sitetype) {
      let param = { responseType: 'blob' }
      http('get', '/sitetypes/export/' + sitetype.id, param, (response) => {
        const url = window.URL.createObjectURL(new Blob([response.data]))
        const link = document.createElement('a')
        link.href = url
        link.setAttribute('download', sitetype.name + '.json')
        document.body.appendChild(link)
        link.click()
      })
    },
    del (id) {
      this.$q.dialog({
        title: '删除场地类型',
        message: '确定删除该场地类型么？',
        ok: {
          label: '删除',
          color: 'red'
        },
        cancel: {
          label: '取消'
        },
        persistent: true
      }).onOk((data) => {
        let delUrl = '/sitetypes/' + id
        http('delete', delUrl, '', (response) => {
          this.refreshList()
        })
      })
    },
    add () {
      let url = '/home/sitetypes/addsitetype'
      this.$router.push({ path: url })
    },
    goto (id) {
      var page = {
        name: 'sitetype',
        params: { id: id }
      }
      this.$router.push(page)
    },
    refresh (done) {
      this.refreshList()
      done()
    },
    refreshList () {
      this.siteTypes = []
      this.$refs.scroll.reset()
      this.$refs.scroll.resume()
      this.$refs.scroll.trigger()
    },
    loadMore (pageNo, done) {
      pageNo--
      let pid = { params: { currentPage: pageNo } }

      http('get', '/sitetypes/page', pid, (response) => {
        done()
        if (response.data.totalElements > 0) {
          this.siteTypes.push(...response.data.content)
          if (response.data.last) {
            this.$refs.scroll.stop()
          }
        } else {
          this.$refs.scroll.stop()
        }
      })
    }
  }
}
</script>

<style>
</style>
