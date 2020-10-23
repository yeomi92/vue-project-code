<template>
  <div>
    <select-work-unit
      v-if="showPopup"
      @popupClose="showPopup = false"
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
          v-model="workUnit"
          :placeholder="!!workType.value ? '광고매체를 선택해주세요' : '작업을 먼저 선택해주세요'"
          disabled
        />
        <click-button
          :text="`광고매체 ${!!workUnit ? '재' : ''}선택`"
          :bgColor="!!workUnit ? 'point-red' : 'base'"
          @click="showPopup = true"
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
          :disabled="!workUnit"
          notSelectedText="광고매체를 먼저 선택해주세요."
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
          notSelectedText="캠페인 유형을 먼저 선택해주세요."
        />
      </div>
    </input-row>
  </div>
</template>

<script>
import SelectWorkUnit from './inputBasicInfo/SelectWorkUnit'
export default {
  name: 'DaUploadDaUploadSubpopupInputBasicInfo',
  components: {
    SelectWorkUnit
  },
  data () {
    return {
      type: {
        value: 'creative',
        options: [{
          title: '소재 대량작업',
          description: '소재 파일을 대량으로 업로드하여 등록/수정/삭제하는 작업입니다.',
          value: 'creative',
          iconContent: 'folder',
          hide: false
        }]
      },
      workUnit: '',
      campaignType: {
        value: '',
        options: [{
          text: '디스플레이 캠페인',
          value: 'display'
        }, {
          text: '비즈보드 캠페인',
          value: 'bizboard'
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
      showPopup: false
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
