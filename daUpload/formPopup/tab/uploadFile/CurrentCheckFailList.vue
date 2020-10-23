<template>
  <sub-popup
    title="최근 검수 실패"
    @popupClose="popupClose">
    <div class="common__list-layout">
      <div class="common__list-layout__header">
        최근 검수 실패 : {{failImageList.length}}개
      </div>
      <div class="common__list-layout__section">
        <list-table
          indexKey="id"
          :data="list"
          :showRowNum="10"
        >
          <template v-slot="{ row }">
            <list-table-column label=" " :width="50" align="center">
              {{ list.length - row.index }}
            </list-table-column>
            <list-table-column label="이미지 파일명" :width="400">
              {{ row.name }}
            </list-table-column>
            <list-table-column label="실패 사유" width="auto">
              {{ row.msg }}
            </list-table-column>
          </template>
        </list-table>
      </div>
    </div>
    <template v-slot:footer>
      <click-button text="확인" @click="popupClose" />
    </template>
  </sub-popup>
</template>

<script>
export default {
  name: 'DaUploadDaUploadSubpopupUploadFileCurrentCheckFailList',
  props: ['failImageList'],
  methods: {
    popupClose () {
      this.$emit('popupClose')
    }
  },
  computed: {
    list () {
      return this.failImageList.map((i, index) => {
        return {
          ...i,
          index
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
>>> .sub-popup__body {
  padding: 15px !important;
  min-width: auto !important;
}
</style>
