<template>
  <sub-popup
    title="광고매체 선택"
    @popupClose="popupClose"
  >
    <list-table
      indexKey="id"
      :data="workUnitList"
      :showRowNum="10"
      useRowButton="radio"
    >
      <template v-slot="{ row }">
        <list-table-column label="광고매체" width="300">
          {{ row.name }}
        </list-table-column>
        <list-table-column label="매체" width="150">
          {{ row.media_name }}
        </list-table-column>
        <list-table-column label="광고상품" width="150">
          {{ row.ad_product_name }}
        </list-table-column>
        <list-table-column label="광고유형" width="80" align="center">
          {{ row.ad_type_name }}
        </list-table-column>
      </template>
      <template v-slot:noData>
        <div>
          선택 가능한 광고매체가 없습니다.
        </div>
      </template>
    </list-table>
    <template v-slot:footer>
      <click-button text="선택" @click="selectWorkUnit" />
      <click-button text="취소" bgColor="grey" @click="popupClose" />
    </template>
  </sub-popup>
</template>

<script>
import { mapState, mapActions } from 'vuex'
export default {
  name: 'DaUploadDaUploadSubpopupInputBasicInfoSelectWorkUnit',
  data () {
    return {
      workUnitList: []
    }
  },
  props: ['selectedWorkUnit'],
  computed: {
    ...mapState({
      advId: (state) => state.advInfo.adv_id
    })
  },
  async mounted () {
    const {
      success,
      workUnitList
    } = await this.getWorkUnitList({
      params: {
        adv_id: this.advId,
        app_code: 'APP032',
        use_paging: false,
        is_account_ser: true
      }
    })
    if (success) {
      this.workUnitList = workUnitList.map(i => {
        if (i.id === this.selectedWorkUnit.id) {
          i.checked = true
        }
        return i
      })
    }
  },
  methods: {
    ...mapActions([
      'getWorkUnitList'
    ]),
    selectWorkUnit () {
      const workUnit = this.workUnitList.find(workUnit => workUnit.checked)
      if (!workUnit) {
        this.$dialog({
          title: 'DA 업로드 작업',
          mainText: '광고매체를 선택해주세요.',
          type: 'alert'
        })
        return
      }
      this.$emit('select', workUnit)
      this.popupClose()
    },
    popupClose () {
      this.$emit('popupClose')
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
