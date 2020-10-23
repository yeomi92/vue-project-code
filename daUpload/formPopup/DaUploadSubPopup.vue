<template>
  <sub-popup
    title="DA 업로드 작업"
    @popupClose="popupClose"
  >
    <full-width-tab
      :useClick="false"
      :activeIndex="currentStep"
      :tabs="steps"
      :showTabIndex="true">
      <div class="da-upload-form-popup">
        <div v-show="isFistStep">
          <select-work-unit
            v-if="showWorkUnitPopup"
            :selectedWorkUnit="workUnit"
            @select="(value) => workUnit = value"
            @popupClose="showWorkUnitPopup = false"
          />
          <input-row rowName="작업 구분">
            <div class="row-description-text">
              등록하고자 하는 작업을 선택하세요.
            </div>
            <icon-button-group
              v-model="type.value"
              :options="type.options"
            />
          </input-row>
          <input-row rowName="광고매체 선택">
            <div class="row-description-text">
              파일을 등록하고자 하는 광고 매체를 선택하세요.
            </div>
            <div class="inline-row">
              <input-field-v2
                class="work-unit-input-field"
                v-model="workUnit.name"
                :placeholder="!!type.value ? '광고매체를 선택해주세요' : '작업을 먼저 선택해주세요'"
                :disabled="true"
              />
              <click-button
                :text="`광고매체 ${!!workUnit.name ? '재' : ''}선택`"
                :bgColor="!!workUnit.name ? 'point-red' : 'base'"
                :disabled="!type.value"
                @click="showWorkUnitPopup = true"
              />
            </div>
          </input-row>
          <input-row rowName="캠페인유형 선택">
            <div class="row-description-text">
              작업하실 캠페인의 유형을 선택하세요.
            </div>
            <div class="inline-row">
              <drop-down
                v-model="campaignType.value"
                :options="campaignType.options"
                fixedWidth="230"
                :disabled="!workUnit.name"
                :notSelectedText="!workUnit.name ? '광고매체를 먼저 선택해주세요.' : '캠페인 유형을 선택해주세요.'"
              />
            </div>
          </input-row>
          <input-row rowName="작업유형 선택">
            <div class="row-description-text">
              등록하실 작업의 유형을 선택하세요.
            </div>
            <div class="inline-row">
              <drop-down
                v-model="workType.value"
                :options="workType.options"
                fixedWidth="230"
                :disabled="!campaignType.value"
                :notSelectedText="!campaignType.value ? '캠페인 유형을 먼저 선택해주세요.' : '작업유형을 선택해주세요.'"
              />
            </div>
          </input-row>
        </div>
        <div v-show="!isFistStep">
          <current-check-fail-list
            v-if="showFailListPopup"
            :failImageList="failImageList"
            @popupClose="showFailListPopup = false"
          />
          <direct-input
            v-if="showDirectInputPopup"
            :objectCode="campaignType.value"
            :searchValue="type.value"
            :workType="workType.value"
            @popupClose="showDirectInputPopup = false"
            @setUploadFile="setUploadFile"
          />
          <input-row
            v-if="workType.value !== 'del' && type.value === 'creative'"
            rowName="이미지 업로드"
            :tooltip="{
              text: `내 컴퓨터에서 업로드 할 이미지를 선택하면,<br>
                    등록 가능한 형식인지 검수합니다.<br>
                    검수 완료된 이미지는 하단의 파일 업로드 영역에서<br>
                    나머지 정보와 함께 등록되어야 최종적으로 완료됩니다.`
            }"
          >
            <div class="row-description-text">
              [이미지 파일 찾기] 버튼을 눌러 소재 이미지를 등록해주세요.
            </div>
            <div class="row-sub-description-text">
              이미지 검수 완료 후 성공한 결과만 리스트에 나타납니다.<br>
              검수에 실패한 파일이 존재할 경우, 최근 검수 실패를 클릭하시면, 자세한 내용을 확인하실 수 있습니다.<br>
              (최근 작업의 실패 내역만 나타납니다.)
            </div>
            <div class="common__list-layout">
              <div class="common__list-layout__header">
                <click-button
                  text="이미지 파일 찾기"
                  bgColor="blue"
                  :fileButton="true"
                  :fileButtonOption="{
                    multiple: true,
                    accept: '.png, .jpg, .jpeg'
                  }"
                  :disabled="imageList.length === 250"
                  @selectFile="selectImageFile"
                />
                <click-button
                  text="제거"
                  bgColor="point-red"
                  :disabled="checkedImageNameSet.size === 0"
                  @click="removeImage"
                />
                <div class="common__list-layout__header--right-push" v-if="imageList.length !== 0 || failImageList.length !== 0">
                  최근 검수 실패 : <span class="underline-text" :class="isEmptyfailImageList ? '' : 'use-click'" @click="showFailImageListPopup">{{failImageList.length}}개</span>
                </div>
              </div>
              <div class="common__list-layout__section">
                <list-table
                  indexKey="id"
                  :data="imageList"
                  :showRowNum="3"
                  useRowButton="checkbox"
                >
                  <template v-slot="{ row }">
                    <list-table-column :label="`검수 성공 이미지 파일${isEmptyimageList ? '' : `(총 ${imageList.length}개)`}`" :width="400" align="center">
                      {{row.name}}
                    </list-table-column>
                    <list-table-column label="이미지 용량" width="auto" align="center">
                      {{row.size/1000}}KB
                    </list-table-column>
                  </template>
                  <template v-slot:noData>
                    <div>
                      이미지 파일을 선택해주세요.
                    </div>
                  </template>
                </list-table>
              </div>
            </div>
          </input-row>
          <input-row
            class="upload-file-section"
            rowName="파일 업로드"
            :tooltip="{
              text: `이미지 외에 나머지 소재 정보를 업로드합니다.<br>
                    이미지가 먼저 업로드 완료된 후에 이용 가능하며,<br>
                    엑셀 파일 혹은 직접 입력하기 기능을 통해 등록할 수 있습니다.`
            }"
          >
            <grid-flex align="center">
              <input-radios
                v-model="uploadType"
                :options="[
                  { text: '엑셀 업로드하기', value: 'excel' },
                  { text: '직접 입력하기', value: 'direct' }
                ]"
                :block="false"
                gap="40"
              />
              <click-button
                v-if="uploadType === 'direct'"
                style="margin-left: 15px;"
                bgColor="blue"
                :text="'입력'"
                height="30"
                @click="showDirectInputPopup = true"
              />
            </grid-flex>
            <div class="row-description-text" v-if="uploadType === 'excel'">
              템플릿 파일을 다운로드 하신 후, [파일 찾기] 버튼을 눌러 업로드 파일을 등록해주세요.
            </div>
            <div class="inline-row" v-if="uploadType === 'excel'">
              <input-field-v2
                class="work-unit-input-field"
                v-model="uploadFile.name"
                :placeholder="!!uploadFile ? '업로드할 파일을 선택해주세요.' : '작업을 먼저 선택해주세요'"
                disabled
              />
              <click-button
                text="파일 찾기"
                bgColor="blue"
                :fileButton="true"
                @selectFile="selectFile"
              />
              <click-button
                text="템플릿 파일 다운"
                @click="templateDownload"
              />
            </div>
            <div v-if="isEmptyimageList" class="upload-file-section__mask">이미지 업로드 완료 후, 작업을 진행할 수 있습니다.</div>
          </input-row>
        </div>
      </div>
    </full-width-tab>
    <div class="common__footer-btn">
      <click-button
        :text="isFistStep ? '다음' : '이전'"
        :bgColor="isFistStep ? 'base' : 'white'"
        @click="changeStep"
      />
      <click-button
        v-if="!isFistStep"
        text="등록"
        @click="save"
      />
      <click-button
        text="취소"
        bgColor="grey"
        @click="popupClose"
      />
    </div>
  </sub-popup>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import SelectWorkUnit from './tab/inputBasicInfo/SelectWorkUnit'
import CurrentCheckFailList from './tab/uploadFile/CurrentCheckFailList'
import DirectInput from './tab/uploadFile/DirectInput'
import { UPLOAD_CODES } from '@@codes'
import Upload from '@/mixins/Upload'
import Download from '@/mixins/Download'
import UUID from 'uuid/v4'
// import inputBasicInfo from './tab/InputBasicInfo'
// import UploadFile from './tab/UploadFile'
const DISPLAY_TYPE_LIST = ['jpg', 'jpeg', 'png']
const BIZ_TYPE_LIST = ['png']
const LIMIT_COUNT = 250
const DISPLAY_IMAGE_SIZE = {
  300: {
    300: 200
  },
  640: {
    480: 400,
    200: 100
  },
  1200: {
    600: 150,
    628: 400
  },
  500: {
    500: 300
  },
  600: {
    500: 300
  },
  1310: {
    240: 300
  }
}
const BIZ_IMAGE_SIZE = {
  1029: {
    222: 300
  }
}
export default {
  name: 'DaUploadSubPopup',
  components: {
    // inputBasicInfo,
    // UploadFile,
    SelectWorkUnit,
    CurrentCheckFailList,
    DirectInput
  },
  mixins: [
    Upload,
    Download
  ],
  props: ['reload'],
  data () {
    return {
      test: undefined,
      currentStep: 0,
      steps: [{
        text: '기본정보 입력'
      }, {
        text: '파일 업로드'
      }],
      // step1
      type: {
        value: undefined,
        options: [{
          title: '소재 대량작업',
          description: '소재 파일을 대량으로 업로드하여 등록/수정/삭제하는 작업입니다.',
          value: 'creative',
          iconContent: 'folder',
          hide: false
        }, {
          title: '소재 상태 및 입찰가 변경',
          description: '소재 파일을 대량으로 업로드하여 소재의 상태 혹은 입찰가를 변경하는 작업입니다.',
          value: 'creative_status_bidamt',
          iconContent: 'creative-status-bidamt',
          hide: false
        }]
      },
      workUnit: {
        name: undefined
      },
      campaignType: {
        value: '',
        options: [{
          text: '디스플레이 캠페인',
          value: 'DISPLAY'
        }, {
          text: '비즈보드 캠페인',
          value: 'TALK_BIZ_BOARD'
        }]
      },
      workType: {
        value: '',
        options: [{
          text: '등록',
          value: 'new'
        }, {
          text: '수정',
          value: 'mod'
        }, {
          text: '삭제',
          value: 'del'
        }]
      },
      showWorkUnitPopup: false,
      // step2
      imageList: [],
      failImageList: [],
      uploadType: 'excel',
      uploadFile: {
        name: '',
        size: 0,
        file: null
      },
      uploadBucket: {
        file: undefined,
        image: []
      },
      showDirectInputPopup: false,
      showFailListPopup: false,
      uuid: UUID()
    }
  },
  computed: {
    ...mapState({
      advId: (state) => state.advInfo.adv_id
    }),
    isFistStep () {
      return this.currentStep === 0
    },
    checkedImageNameSet () {
      const list = this.imageList.filter(i => i.checked)
      return new Set(list.map(({ name }) => name))
    },
    completeStep1 () {
      return this.workType.value && this.campaignType.value && this.workUnit.name
    },
    imageNameSet () {
      return new Set(this.imageList.map(({ name }) => name))
    },
    isEmptyimageList () {
      return this.workType.value === 'new' && this.imageList.length === 0 && this.type.value === 'creative'
    },
    isEmptyfailImageList () {
      return this.failImageList.length === 0
    }
  },
  watch: {
    'workUnit.id' () {
      this.campaignType.value = ''
    },
    'campaignType.value' (value) {
      this.workType.value = ''
    },
    'workType.value' () {
      this.imageList = []
      this.failImageList = []
      this.uploadType = 'excel'
      this.uploadFile = {
        name: '',
        size: 0,
        file: null
      }
    },
    'type.value' (value) {
      this.workUnit = {
        name: undefined
      }
      this.campaignType.value = ''
      this.workType.value = ''
      if (value === 'creative') {
        this.workType.options = [{
          text: '등록',
          value: 'new'
        }, {
          text: '수정',
          value: 'mod'
        }, {
          text: '삭제',
          value: 'del'
        }]
      } else {
        this.workType.options = [{
          text: '수정',
          value: 'mod'
        }]
      }
    }
  },
  methods: {
    ...mapActions([
      'createDaMediaWizardMng',
      'getTemplateInfo'
    ]),
    // common
    popupClose (success) {
      this.$dialog({
        title: 'DA 업로드 작업',
        mainText: '작업을 취소하시면 입력한 내용은 저장되지 않습니다.<br>취소하시겠습니까?',
        type: 'comfirm',
        okCallback: () => {
          this.$emit('popupClose', success)
        }
      })
    },
    changeStep () {
      // step1에서 입력을 다 하지 않은 경우
      if (this.isFistStep) {
        if (!this.type.value) {
          this.$dialog({
            title: 'DA 업로드 작업',
            mainText: '다음 단계로 이동할 수 없습니다.',
            subText: '작업 구분을 선택해주세요.',
            type: 'alert'
          })
          return
        }
        if (!this.workUnit.name) {
          this.$dialog({
            title: 'DA 업로드 작업',
            mainText: '다음 단계로 이동할 수 없습니다.',
            subText: '광고매체를 선택해주세요.',
            type: 'alert'
          })
          return
        }
        if (!this.campaignType.value) {
          this.$dialog({
            title: 'DA 업로드 작업',
            mainText: '다음 단계로 이동할 수 없습니다.',
            subText: '캠페인유형을 선택해주세요.',
            type: 'alert'
          })
          return
        }
        if (!this.workType.value) {
          this.$dialog({
            title: 'DA 업로드 작업',
            mainText: '다음 단계로 이동할 수 없습니다.',
            subText: '작업유형을 선택해주세요.',
            type: 'alert'
          })
          return
        }
      }
      this.currentStep = this.currentStep + (this.isFistStep ? 1 : -1)
    },
    async save () {
      if (this.isEmptyimageList) {
        this.$dialog({
          title: 'DA 업로드 작업',
          mainText: '업로드 작업을 진행 할 수 없습니다.',
          subText: '검수 성공한 이미지 파일이 없어 업로드 작업을 진행할 수 없습니다.',
          type: 'alert'
        })
        return
      }
      if (this.uploadFile.file === null) {
        this.$dialog({
          title: 'DA 업로드 작업',
          mainText: '업로드 작업을 진행 할 수 없습니다.',
          subText: '작업파일을 등록하지 않아 업로드 작업을 진행할 수 없습니다.',
          type: 'alert'
        })
        return
      }
      // const uploadImage = this.type.value === 'creative' && this.workType.value === 'new'
      const { image, file } = this.uploadBucket
      const emptyImage = image.length === 0
      const params = {
        type: this.type.value,
        media_code: this.workUnit.media_code,
        work_unit_id: this.workUnit.id,
        work_unit_name: this.workUnit.name,
        campaign_type: this.campaignType.value,
        work_type: this.workType.value,
        account_ser: this.workUnit.account_ser,
        upload_file: {
          bucket_name: file.bucket_name,
          object_name: file.object_name
        },
        upload_img_info: {
          bucket_name: emptyImage ? '' : image[0].bucket_name,
          object_file_path: emptyImage ? '' : image[0].file_path,
          object_file_name_list: this.imageNameSet || ''
        }
      }
      const {
        success
      } = await this.createDaMediaWizardMng({
        pathVariable: {
          id: this.$route.params.id
        },
        params
      })
      if (success) {
        this.reload()
        this.$emit('popupClose', success)
      }
    },
    // step1
    // step2
    // 이미지 파일 찾기
    async selectImageFile ({ files }) {
      const collator = new Intl.Collator('en', { numeric: true, sensitivity: 'base' })
      const sortFiles = Array.from(files).sort((a, b) => collator.compare(a.name, b.name))
      // 갯수 체크
      if (this.imageList.length + files.length > LIMIT_COUNT) {
        this.$dialog({
          title: 'DA 업로드 작업',
          mainText: '새로운 소재 이미지를 등록할 수 없습니다.',
          subText: `이미지는 한 작업 당 최대 ${LIMIT_COUNT}개 까지 가능합니다.<br>선택한 이미지 개수를 조정해 주세요.`,
          type: 'alert'
        })
        return
      }
      // campaignType 체크
      const isDisplayCampaign = this.campaignType.value === 'DISPLAY'
      const typeList = isDisplayCampaign ? DISPLAY_TYPE_LIST : BIZ_TYPE_LIST
      const imageSize = isDisplayCampaign ? DISPLAY_IMAGE_SIZE : BIZ_IMAGE_SIZE
      const successImageList = []
      const failImageList = []
      const promise = sortFiles.map(async (file, index) => {
        // 동일이름 체크
        if (this.imageNameSet.has(file.name)) {
          failImageList.push({
            name: file.name,
            file,
            msg: '파일명이 중복됩니다.'
          })
        } else {
          const { width, height } = await this.getImageWidthAndHeight(file)
          const { name, size, type } = file
          // 확장자 체크
          if (typeList.includes(type.split('/')[1])) {
            // 사이즈 체크
            if (Object.keys(imageSize).includes(String(width)) && Object.keys(imageSize[width]).includes(String(height))) {
              // 용량 체크
              if (imageSize[width][height] >= (file.size / 1000)) {
                successImageList.push({
                  name, size, file
                })
              } else {
                failImageList.push({
                  name: file.name,
                  file,
                  msg: '업로드 가능한 용량을 초과하였습니다.'
                })
              }
            } else {
              failImageList.push({
                name: file.name,
                file,
                msg: '업로드 가능한 이미지 사이즈가 아닙니다.'
              })
            }
          } else {
            failImageList.push({
              name: file.name,
              file,
              msg: '업로드 가능한 포맷이 아닙니다.'
            })
          }
        }
      })
      await Promise.all(promise)
      if (successImageList.length > 0) {
        this.imageList.push(...Array.from(successImageList).sort((a, b) => collator.compare(a.name, b.name)))
        const res = await this.$_s3Upload(UPLOAD_CODES.DA_UPLOAD_IMG, { data: files }, { uuid: this.uuid }, true)
        this.uploadBucket.image.push(...res)
      }
      if (failImageList.length > 0) {
        this.failImageList.push(...failImageList)
        this.$dialog({
          title: 'DA 업로드 작업',
          mainText: `<span class="red-text">* 선택한 이미지 중 업로드할 수 없는 이미지가 있습니다.<br>(실패한 이미지 개수 : 총 ${failImageList.length}개)</span>`,
          subText: '이미지 리스트 우측 상단의 [최근 검수 실패]를 클릭하여,<br>사유를 확인해 주세요.',
          type: 'alert'
        })
      }
    },
    getImageWidthAndHeight (file) {
      return new Promise((resolve, reject) => {
        const img = new Image()
        img.src = URL.createObjectURL(file)
        img.onload = () => {
          resolve(img)
        }
      })
    },
    // 파일 직접 업로드
    async selectFile ({ files }) {
      const file = files[0]
      const dialogState = { title: 'DA 업로드 작업', mainText: '파일을 추가할 수 없습니다.', subText: '', type: 'alert' }
      if (!file.name.endsWith('.csv')) {
        dialogState.subText = `CSV 파일만 업로드 가능합니다. 재선택 해주세요.`
        this.$dialog(dialogState)
        return
      }
      this.uploadFile = {
        name: file.name,
        size: file.size,
        file: file
      }
      this.uploadReportFile()
    },
    showFailImageListPopup () {
      if (this.isEmptyfailImageList) {
        return
      }
      this.showFailListPopup = true
    },
    removeImage () {
      if (this.checkedImageNameSet.size === 0) {
        this.$dialog({
          title: 'DA 업로드 작업',
          mainText: '제거할 이미지를 선택해 주세요.',
          type: 'alert'
        })
        return
      }
      this.imageList = this.imageList.filter(i => !this.checkedImageNameSet.has(i.name))
    },
    setUploadFile (file) {
      this.uploadFile = {
        name: file.name,
        size: file.size,
        file: file
      }
      this.uploadReportFile()
    },
    async uploadReportFile () {
      const res = await this.$_s3Upload(UPLOAD_CODES.DA_UPLOAD_REPORT, { data: this.uploadFile.file }, { type: this.type.value, workType: this.workType.value })
      this.uploadBucket.file = res[0]
    },
    async templateDownload () {
      const {
        url
      } = await this.getTemplateInfo({
        params: {
          adv_id: this.advId,
          app_code: 'APP032',
          object_level: 'campaign_type',
          object_code: this.campaignType.value, // 캠페인 타입(DISPLAY / TALK_BIZ_BOARD)
          search_field: 'media_wizard_mng_type',
          search_value: this.type.value, // 작업 구분(creative / creative_status_bidamt)
          work_type: this.workType.value, // 작업 유형(new / mod / del)
          is_upload: 1,
          upload_type: 'excel'
        }
      })
      this.$_aDownload(`//${url}`, 'zero_frame')
    }
  }
}
</script>

<style lang="scss" scoped>
>>> .sub-popup__body {
  padding: 0;
  min-width: 775px;
}
.da-upload-form-popup {
  padding: 15px;
  >>> .row-description-text {
    margin: 13px 0 10px 0;
  }
  >>> .icon-button-group {
    width: 600px;
  }
  >>> .inline-row {
    display: flex;
    .work-unit-input-field {
      flex: 1;
    }
    &--file {
      .click-button {
        height: 30px;
      }
      .remain-file-size {
        margin-left: auto;
        align-self: flex-end;
      }
      margin-bottom: 10px;
    }
  }
  .common__list-layout {
    padding: 10px 0;
  }
  .row-sub-description-text {
    font-size: 12px;
    line-height: 15px;
  }
  .underline-text {
    color: $pointRed;
    text-decoration: underline;
    margin-left: 5px;
    &.use-click {
      cursor: pointer;
    }
  }
  .upload-file-section {
    position: relative;
    &__mask {
      position: absolute;
      top: 0;
      left: 115px;
      background-color: #fff;
      opacity: 0.9;
      width: calc(100% - 115px);
      height: 120px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
    }
  }
}
</style>
