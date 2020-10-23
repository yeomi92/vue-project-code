<template>
  <section-wrapper
    :mainTitle="'DA 업로드 작업'"
    guideLink="''"
  >
    <div class="common__list-layout">
      <div class="common__list-layout__header">
        <click-button
          v-if="canRegist"
          text="신규작업"
          bgColor="blue"
          @click="openDaUploadForm"
        />
        <div class="common__list-layout__header--right-push">
          <input-field-v2
            v-model="searchedText"
            suffix="search-icon"
            placeholder="작업명"
            @enter="$_changeRouteQuery('name', searchedText)"
          />
          <click-button
            :text="'필터초기화'"
            @click="$_filterReset"
            appearance="filter"
          />
          <click-button
            :text="'새로고침'"
            @click="$_listReload"
            appearance="refresh"
          />
        </div>
      </div>
      <div class="common__list-layout__section">
        <list-table
          indexKey="id"
          :data="list"
          :showRowNum="pagingData.row_num"
          :isSearched="!!pagingData.name"
        >
          <template v-slot="{ row }">
            <list-table-column label="작업명" width="auto">
              <a class="list-table--clickable-text" @click="openDaUploadForm(row.id)">{{row.name}}</a>
            </list-table-column>
            <list-table-column label="업로드 현황" align="center"
              tooltipTitle="업로드 현황"
              tooltipText="업로드 현황은 업로드 진행 상태별로 세부 작업 건수를 나타내는 현황표입니다.<br>
                           각 상태별로 현재 몇 건의 세부 작업 업로드가 진행되고 있는지 확인하실 수 있습니다.">
              <list-table-column label="전체" :width="80" align="center">
                {{row.total_cnt}}
              </list-table-column>
              <list-table-column label="대기" :width="80" align="center">
                {{row.ready_cnt}}
              </list-table-column>
              <list-table-column label="진행중" :width="80" align="center">
                {{row.working_cnt}}
              </list-table-column>
              <list-table-column label="성공" :width="80" align="center">
                {{row.success_cnt}}
              </list-table-column>
              <list-table-column label="부분성공" :width="80" align="center">
                {{row.partial_success_cnt}}
              </list-table-column>
              <list-table-column label="실패" :width="80" align="center">
                {{row.fail_cnt}}
              </list-table-column>
            </list-table-column>
            <list-table-column label="등록자" :width="150">
              {{row.created_name_by}}
            </list-table-column>
            <list-table-column label="최근 수정시간" :width="180">
              <span v-html="row.updated_at ? row.updated_at : '-'" />
            </list-table-column>
          </template>
          <template v-slot:noData>
            <div class="no-data" v-if="isSearched">
              <h1>검색 조건과 일치하는 데이터가 없습니다.</h1>
              <div>- 단어의 철자가 정확한지 확인해 주세요.</div>
              <div>- 다른 검색어로 검색해 보세요.</div>
              <div>- 상단의 필터를 변경해 보세요.</div>
            </div>
            <div v-else>
              데이터가 없습니다.
            </div>
          </template>
        </list-table>
        <paging
          v-if="list.length > 0"
          :page_num="pagingData.page_num"
          :total_page_num="pagingData.total_page_num"
          :row_num="pagingData.row_num" />
      </div>
    </div>
  </section-wrapper>
</template>

<script>
import { mapGetters, mapState, mapActions } from 'vuex'
import ListMixin from '@/mixins/ListMixin'
import { PAGE } from '@@routes'
export default {
  name: 'OperatingDaUploadList',
  mixins: [
    ListMixin
  ],
  data () {
    return {
      list: [],
      searchedText: '',
      isSearched: false
    }
  },
  computed: {
    ...mapGetters({
      canRegist: 'canRegist',
      canDelete: 'canDelete'
    }),
    ...mapState({
      advId: (state) => state.advInfo.adv_id
    })
  },
  watch: {
    $route: {
      handler (to, from) {
        this.$_setPagingData({
          page_num: !to.params.page ? 1 : Number(to.params.page),
          row_num: !to.query.row_num ? 10 : parseInt(to.query.row_num),
          name: !to.query.name ? null : to.query.name
        })
        this.isSearched = !!to.query.name
        this.searchedText = this.pagingData.name
        this.getList()
      },
      immediate: true
    }
  },
  methods: {
    ...mapActions([
      'getDaMediaWizardUploadList',
      'createDaMediaWizard'
    ]),
    async getList () {
      const {
        success,
        list,
        totlaPageNum
      } = await this.getDaMediaWizardUploadList({
        params: {
          adv_id: this.advId,
          execute_type: 'direct',
          use_paging: true,
          ...this.pagingData,
          name: this.searchedText
        }
      })
      if (success) {
        this.list = list
        this.$_setPagingData({
          total_page_num: totlaPageNum
        })
      }
    },
    async openDaUploadForm (id) {
      if (id === undefined) {
        const {
          success,
          mediaWizardId
        } = await this.createDaMediaWizard({
          params: {
            adv_id: this.advId,
            execute_type: 'direct'
          }
        })
        if (success) {
          console.log(mediaWizardId)
          this.$router.push({
            name: PAGE.DA_UPLOAD_FORM,
            params: {
              id: mediaWizardId
            }
          })
        }
        return
      }
      this.$router.push({
        name: PAGE.DA_UPLOAD_FORM,
        params: {
          id
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.no-data {
  text-align: center;
  h1 {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 20px;
  }
}
</style>
