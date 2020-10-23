<template>
  <sub-popup
    title="직접 입력하기"
    @popupClose="popupClose"
  >
    <spread-sheet-static-column
      v-if="initSpreadSheet"
      ref="spreadsheet"
      :title="title"
      :showColumnNum="showColumnNum"
      :cellWidth="140"
      :keepData="true"
    >
      <template v-slot:description>
        <p v-html="useConfig.DESCRIPTION"></p>
      </template>
    </spread-sheet-static-column>
    <span v-else />
    <template v-slot:footer>
      <click-button
        text="입력 완료"
        bgColor="blue"
        @click="upload"
      />
      <click-button
        text="취소"
        bgColor="grey"
        @click="cancel"
      />
    </template>
  </sub-popup>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import { DIRECT_INPUT } from '@/constants/DirectInput'

export default {
  name: 'SAUploadFormDirectInputSubPopup',
  props: ['objectCode', 'searchValue', 'workType'],
  data () {
    return {
      title: [],
      fileDescription: '',
      showColumnNum: 0
    }
  },
  computed: {
    ...mapState({
      advId: (state) => state.advInfo.adv_id
    }),
    initSpreadSheet () {
      return this.objectCode && this.searchValue && this.workType && this.title.length > 0
    },
    useConfig () {
      const rootUseConfig = DIRECT_INPUT.KAKAOMOMENT[this.searchValue]
      let useConfig = rootUseConfig
      useConfig = useConfig[this.objectCode] || useConfig
      return useConfig[this.workType] ? useConfig[this.workType] : rootUseConfig[this.workType]
    }
  },
  mounted () {
    this.getDescription()
  },
  methods: {
    ...mapActions([
      'getTemplateInfo'
    ]),
    async getDescription () {
      const {
        description,
        fieldKorName
      } = await this.getTemplateInfo({
        params: {
          adv_id: this.advId,
          app_code: 'APP032',
          object_level: 'campaign_type',
          object_code: this.objectCode, // 캠페인 타입(DISPLAY / TALK_BIZ_BOARD)
          search_field: 'media_wizard_mng_type',
          search_value: this.searchValue, // 작업 구분(creative / creative_status_bidamt)
          work_type: this.workType, // 작업 유형(new / mod / del)
          is_upload: 1,
          upload_type: 'direct'
        }
      })

      const requiredColumn = this.useConfig.REQUIRED_COLUMN
      this.fileDescription = description
      const title = fieldKorName.map(name => {
        return {
          text: name,
          required: requiredColumn.indexOf(name) > -1
        }
      })

      this.showColumnNum = title.length > 6 ? 6 : title.length
      this.title = title
    },
    upload () {
      const file = this.$refs.spreadsheet.getUploadFile(this.fileDescription)
      if (file === undefined) {
        return
      }
      this.$emit('setUploadFile', file)
      this.popupClose()
    },
    cancel () {
      if (!this.$refs.spreadsheet.checkedEmpty()) {
        this.$dialog({
          title: '직접 입력하기',
          mainText: '직접 입력하기를 취소하시겠습니까?',
          type: 'comfirm',
          okCallback: () => {
            this.popupClose()
          }
        })
        return
      }
      this.popupClose()
    },
    popupClose () {
      this.$emit('popupClose')
    }
  }
}
</script>
<style lang="scss" scoped>
.footer {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}
>>> .sub-popup__body {
  padding: 15px !important;
  min-width: auto !important;
}
</style>
