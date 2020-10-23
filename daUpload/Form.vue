<template>
  <section-wrapper
    mainTitle="DA 업로드 작업"
    :rightTitle="`등록자: ${authInfo.user_name}(${authInfo.login_id}) | 최근 수정시간: ${modifiedDate}`">
    <da-upload-sub-popup
      v-if="popupShow"
      :reload="getList"
      @popupClose="popupShow = false"/>
    <div class="da-upload-form">
      <input-row
        rowName="작업명">
        <div class="da-upload-form__work-name">
          <div class="da-upload-form__work-name--name" v-if="!modifyWorkName">{{workName}}</div>
          <input-field-v2
            v-if="modifyWorkName"
            v-model="workNameInput"
            placeholder="작업명을 입력해주세요."
            :maxlength="18"
            :lengthCheck="true"
          />
          <click-button
            :text="modifyWorkName ? '저장' : '수정'"
            @click="setWorkName"
          />
          <click-button
            v-if="modifyWorkName"
            text="취소"
            bgColor="grey"
            @click="cancelWorkName"
          />
        </div>
      </input-row>
      <input-row
        rowName="작업 건수">
        <div class="da-upload-form__work-count"><strong>{{list.length}}개</strong></div>
        <div class="da-upload-form__work-count">작업파일 등록 버튼을 눌러 작업 파일과 파일을 업로드할 광고매체를 선택해주세요.</div>
        <div class="common__list-layout">
          <div class="common__list-layout__header">
            <click-button
              v-if="canRegist"
              text="작업파일 등록"
              bgColor="blue"
              @click="popupShow = true"
            />
            <drop-down
              v-model="workType.value"
              :options="workType.options"
              notSelectedText="작업구분"
            />
            <drop-down
              v-model="uploadStatus.value"
              :options="uploadStatus.options"
              notSelectedText="업로드 상태"
            />
            <div class="common__list-layout__header--right-push">
              <click-button
                :text="'필터초기화'"
                appearance="filter"
                @click="filterReset"
              />
              <click-button
                :text="'새로고침'"
                appearance="refresh"
                @click="getList"
              />
            </div>
          </div>
          <div class="common__list-layout__section">
            <list-table
              indexKey="id"
              :data="filteredList"
              :showRowNum="10"
            >
              <template v-slot="{ row }">
                <list-table-column label="작업구분" :width="180" align="center">
                  {{row.media_wizard_mng_description === undefined ? '-' : WORK_GUBUN[row.media_wizard_mng_description.type]}}
                </list-table-column>
                <list-table-column label="광고매체 이름" :width="200" align="center">
                  {{row.media_wizard_mng_description === undefined ? '-' : row.media_wizard_mng_description.work_unit_name}}
                </list-table-column>
                <list-table-column label="캠페인 유형" :width="150" align="center">
                  {{row.media_wizard_mng_description === undefined ? '-' : CAMPAIGN_TYPE[row.media_wizard_mng_description.campaign_type]}}
                </list-table-column>
                <list-table-column label="작업 유형" :width="100" align="center">
                  {{row.media_wizard_mng_description === undefined ? '-' : WORK_TYPE[row.media_wizard_mng_description.work_type]}}
                </list-table-column>
                <list-table-column label="업로드 상태" :width="100" align="center">
                  {{UPLOAD_STATUS[row.status]}}
                </list-table-column>
                <list-table-column label="실행일시" :width="200" align="center">
                  {{row.description === undefined ? '-' : row.description.executed_at || '-'}}
                </list-table-column>
                <list-table-column label="예상 완료일시" :width="200" align="center"
                  tooltipTitle="예상 완료일시"
                  tooltipText="요청하신 작업이 완료되는 예상 일시를 확인할 수 있습니다.<br>
                              작업량이 많을 경우 완료 시간이 다소 소요될 수 있습니다.">
                  {{row.description === undefined ? '-' : row.description.estimated_completion_time || '-'}}
                </list-table-column>
                <list-table-column label="완료일시" :width="200" align="center">
                  {{row.description === undefined ? '-' : row.description.completed_at || '-'}}
                </list-table-column>
                <list-table-column label="이미지<br>개수" :width="100" align="center"
                  tooltipTitle="이미지 개수"
                  tooltipText="정상적으로 업로드 완료된 이미지 개수입니다.<br>
                              업로드에 실패한 이미지는 포함되지 않습니다.">
                  {{row.description === undefined ? '-' : row.description.upload_img_cnt}}
                </list-table-column>
                <list-table-column label="작업 건수" align="center">
                  <list-table-column label="신청" :width="80" align="center">
                    {{row.description === undefined ? '-' : row.description.work_status.all_work_row || 0}}
                  </list-table-column>
                  <list-table-column label="성공" :width="80" align="center">
                    {{row.description === undefined ? '-' : row.description.work_status.success_row || 0}}
                  </list-table-column>
                  <list-table-column label="실패" :width="80" align="center">
                    {{row.description === undefined ? '-' : row.description.work_status.fail_row || 0}}
                  </list-table-column>
                </list-table-column>
                <list-table-column label="결과<br>파일" :width="80" align="center"
                  tooltipTitle="결과파일"
                  tooltipText="광고매체에 업로드 작업을 진행한 결과파일입니다.<br>
                              업로드 작업에 실패하거나 부분성공하였을 경우<br>
                              파일을 다운로드 받아 실패한 부분에 대해 결과정보를 확인하시기 바랍니다.">
                  <download-icon
                    v-if="row.description !== undefined && row.description.download_file.object_name"
                    @click="fileDownload(row.description.download_file)"
                  />
                </list-table-column>
                <list-table-column label="비고" :width="150" align="center"
                  tooltipTitle="비고"
                  tooltipText="비고는 등록하신 작업에 대한 메모 용도로 사용하실 수 있습니다.">
                  <input
                    class="memo-input"
                    :class="{ readonly: !canRegist }"
                    type="text"
                    :value="row.memo"
                    @input="row.memo = $event.target.value"
                    @focusout="saveMngMemo(row)"
                  />
                </list-table-column>
              </template>
              <template v-slot:noData>
                <div>
                  등록된 데이터가 없습니다.
                </div>
              </template>
            </list-table>
          </div>
        </div>
      </input-row>
      <div class="common__footer-btn">
        <click-button
          v-if="list.length === 0 && canDelete"
          :text="'삭제'"
          :bgColor="'point-red'"
          @click="clickDelete" />
        <click-button
          text="목록"
          :bgColor="'grey'"
          @click="clickGoList" />
      </div>
    </div>
  </section-wrapper>
</template>

<script>
import { mapState, mapGetters, mapActions } from 'vuex'
import { PAGE } from '@@routes'
import ListMixin from '@/mixins/ListMixin'
import Download from '@/mixins/Download'
import GoList from '@/mixins/GoList'
import DaUploadSubPopup from './formPopup/DaUploadSubPopup'

const WORK_GUBUN = {
  'creative': '소재 대량작업',
  'creative_status_bidamt': '소재 상태 및 입찰가 변경'
}
const UPLOAD_STATUS = {
  'ready': '대기',
  'before_work': '대기',
  'working': '진행중',
  'success': '성공',
  'partial_success': '부분성공',
  'fail': '실패'
}
const CAMPAIGN_TYPE = {
  'DISPLAY': '디스플레이',
  'TALK_BIZ_BOARD': '비즈보드'
}
const WORK_TYPE = {
  'new': '등록',
  'mod': '수정',
  'del': '삭제'
}
export default {
  name: 'OperatingDaUploadForm',
  mixins: [
    ListMixin,
    Download,
    GoList
  ],
  components: {
    DaUploadSubPopup
  },
  watch: {
    $route: {
      handler (to, from) {
        this.getList()
      },
      immediate: true
    }
  },
  data () {
    return {
      WORK_GUBUN: WORK_GUBUN,
      UPLOAD_STATUS: UPLOAD_STATUS,
      CAMPAIGN_TYPE: CAMPAIGN_TYPE,
      WORK_TYPE: WORK_TYPE,
      popupShow: false,
      modifiedDate: this.$moment().format('YYYY-MM-DD HH:mm'),
      modifyWorkName: false,
      workName: '',
      workNameInput: '',
      workType: {
        value: undefined,
        options: [{
          text: '전체',
          value: undefined
        }, {
          text: '소재 대량작업',
          value: 'creative'
        }, {
          text: '소재 상태 및 입찰가 변경',
          value: 'creative_status_bidamt'
        }]
      },
      uploadStatus: {
        value: undefined,
        options: [{
          text: '전체',
          value: undefined
        }, {
          text: '대기',
          value: 'ready'
        }, {
          text: '진행중',
          value: 'working'
        }, {
          text: '성공',
          value: 'success'
        }, {
          text: '부분성공',
          value: 'partial_success'
        }, {
          text: '실패',
          value: 'fail'
        }]
      },
      list: [],
      originList: []
    }
  },
  computed: {
    ...mapState({
      authInfo: (state) => state.authInfo
    }),
    ...mapGetters({
      canRegist: 'canRegist',
      canDelete: 'canDelete'
    }),
    filteredList () {
      const worType = this.workType.value
      const uploadStatus = this.uploadStatus.value
      return this.list.filter(row => {
        let valid = true
        if (
          (worType && row.media_wizard_mng_description.type !== worType) ||
          (uploadStatus && row.status !== uploadStatus)
        ) {
          valid = false
        }
        return valid
      })
    }
  },
  methods: {
    ...mapActions([
      'deleteDaMediaWizard',
      'updateDaMediaWizardWorkName',
      'getDaMediaWizardMngList',
      'updateDaMediaWizardMngMemo'
    ]),
    async setWorkName () {
      // 수정인 경우
      if (this.modifyWorkName) {
        const {
          success
        } = await this.updateDaMediaWizardWorkName({
          pathVariable: {
            id: this.$route.params.id
          },
          params: {
            name: this.workNameInput
          }
        })
        if (success) {
          this.workName = this.workNameInput
          this.modifyWorkName = this.$moment().format('YYYY-MM-DD HH:mm')
        }
      }
      this.modifyWorkName = !this.modifyWorkName
    },
    cancelWorkName () {
      this.workNameInput = this.workName
      this.modifyWorkName = false
    },
    clickGoList () {
      this.$_goList(PAGE.DA_UPLOAD_LIST)
    },
    async clickDelete () {
      this.$dialog({
        title: 'DA 업로드 작업',
        mainText: '작업 생성을 취소하고 목록화면으로 돌아가시겠습니까?',
        subText: '확인 버튼을 누르시면 현재 작업정보를 저장하지 않고 목록화면으로 이동합니다.',
        type: 'comfirm',
        okCallback: async () => {
          const {
            success
          } = await this.deleteDaMediaWizard({
            pathVariable: {
              id: this.$route.params.id
            }
          })
          if (success) {
            this.$router.replace({
              name: PAGE.DA_UPLOAD_LIST,
              params: {
                page: 1
              }
            })
          }
        }
      })
    },
    async getList () {
      const {
        success,
        list,
        name
      } = await this.getDaMediaWizardMngList({
        pathVariable: {
          id: this.$route.params.id
        },
        params: {
          type: this.workType.value,
          status: this.uploadStatus.value
        }
      })
      if (success) {
        this.workName = name
        this.workNameInput = name
        this.list = list
        this.originList = list
      }
    },
    async saveMngMemo ({ media_wizard_mng_id: mngId, memo }) {
      const {
        success
      } = await this.updateDaMediaWizardMngMemo({
        pathVariable: {
          id: this.$route.params.id,
          mngId
        },
        params: {
          memo
        }
      })
      if (!success) {
        console.error('비고 수정 실패')
      }
    },
    filterReset () {
      this.workType.value = undefined
      this.uploadStatus.value = undefined
    },
    fileDownload ({ bucket_name: bucketName, object_name: objectName }) {
      this.$_aDownload(`//${bucketName}/${objectName}`, 'zero_frame')
    }
  }
}
</script>

<style lang="scss" scoped>
.da-upload-form {
  background-color: #fff;
  padding: 30px;
  margin: 0 auto;
  &__work-name {
    display: flex;
    &--name {
      width: 253px;
      height: 57px;
      line-height: 40px;
      margin-right: 5px;
    }
    .input-field-v2 {
      width: 253px;
    }
  }
  &__work-count {
    height: 40px;
    line-height: 40px;
  }
  .common__list-layout {
    padding: 10px 0;
  }
  .memo-input {
    font-size: $fontSize12;
    height: 30px;
    border: 1px solid lightgrey;
    &.readonly {
      pointer-events: none;
    }
  }
}
</style>
