<template>
  <div class="control-panel">
    <div class="control-panel__block">
      <span class="control-panel__title"> Sorting by: </span>
      <div class="control-panel__sorting">
        <template v-for="item in sortedSelectedColumns">
          <div
              class="control-panel__label"
              :class="{'control-panel__label_active' : sortActiveCol === item.value}"
              @click="setSortActiveCol(item.value)"
          > {{ item.name }} </div>
        </template>
      </div>
    </div>
    <div class="control-panel__block">
      <brand-button
          text="Delete"
          :disabled="!selectedRows.length"
          :recursive="true"
          :delete="true"
          :deleteObject="selectedRows"
      >
        <template
            v-if="selectedRows.length"
            v-slot:append
        >
          <span>  &nbsp; ({{ selectedRows.length}}) </span>
        </template>
      </brand-button>
    </div>
    <div class="control-panel__block">
      <brand-dropdown
          v-model="showRowsValue"
          :label="showingRows.find(x => x.value === showRowsCounter).name"
          ref="rowSelect"
      >
        <template v-slot:dropdown-list>
          <div
              class="brand-dropdown__dropdown-item"
              v-for="item in showingRows"
              :key="item.value"
              @click="showRowSelected(item.value)"> {{ item.name }}
          </div>
        </template>
      </brand-dropdown>
    </div>
    <div class="control-panel__block control-panel__block_pagination">
      <brand-button
          text=""
          :secondary="true"
          :squared="true"
          :disabled="!showRowsValue"
          @click="paginationBtnClicked('prev')"
      >
        <template v-slot:append>
          <svg-icon name="chevron-down" class="brand-button__icon brand-button__icon_left">
          </svg-icon>
        </template>
      </brand-button>
      <span class="control-panel__text">
            <strong> {{ showRowsValue + 1}} - </strong>
            <strong v-if="!maxValueReachedFlag"> {{ showRowsValue + showRowsCounter + ' '}} </strong>
            <strong v-else> {{ tableDataLength + ' '}} </strong>
            <span> of </span>
            <strong> {{ ' ' + tableDataLength }} </strong>
          </span>
      <brand-button
          text=""
          :secondary="true"
          :squared="true"
          :disabled="maxValueReachedFlag"
          @click="paginationBtnClicked('next')"
      >
        <template v-slot:append>
          <svg-icon name="chevron-down" class="brand-button__icon brand-button__icon_right">
          </svg-icon>
        </template>
      </brand-button>
    </div>
    <div class="control-panel__block">
      <brand-dropdown :label="sortedSelectedColumns.length + ' columns selected'">
        <template v-slot:dropdown-list>
          <div class="brand-dropdown__dropdown-item brand-dropdown__no-padding brand-dropdown__all">
            <brand-checkbox
                class="brand-dropdown__checkbox"
                :class="{'brand-checkbox_part' : getPartCheckboxState()}"
                :checked="true"
                :object="{name: 'Select all', value: 'all'}"
                @click="allColumnSelected($event, ...arguments)"
                ref="selectAllCheckbox"
            />
          </div>
          <div
              class="brand-dropdown__dropdown-item brand-dropdown__no-padding"
              v-for="item in productParameters"
              :key="item.value"
          >
            <brand-checkbox
                class="brand-dropdown__checkbox"
                ref="showRowsCheckbox"
                :checked="true"
                :object="item"
                @click="columnSelected($event, ...arguments)"
            />
          </div>
        </template>
      </brand-dropdown>
    </div>
  </div>
</template>
<script>
  import { createNamespacedHelpers } from "vuex"
  const { mapState } = createNamespacedHelpers('table');

  export default {
    props: {
      tableDataLength: {
        type: Number,
        required: true,
      },
      baseProductParameters: {
        type: Array,
        required: true
      }
    },
    data() {
      return {
        ...mapState(['sortActiveCol']),
        showingRows: [
          {
            name: '10 Per Page',
            value: 10
          },
          {
            name: '15 Per Page',
            value: 15
          },
          {
            name: '20 Per Page',
            value: 20
          },
        ],
        selectedColumns: this.baseProductParameters.map((item, index) => {
          return {
            ...item,
            basePosition: index + 1,
            tablePosition: index + 1
          }
        }),
      }
    },
    watch: {
      sortActiveCol: function (newVal) {
        this.$store.dispatch('table/setActiveSortCol', newVal);

        this.selectedColumns = this.selectedColumns.map(x => {
          return {
            ...x,
            tablePosition: x.basePosition,
          }
        });

        if (newVal) {
          this.selectedColumns.find(x => x.value === newVal).tablePosition = 0;
        }
      }
    },
    computed: {
      ...mapState([
        'selectedRows',
        'showRowsValue',
        'showRowsCounter',
        'maxValueReachedFlag',
      ]),
      ...mapState({
        tempShowPage: 'showRowsStartValue'
      }),
      productParameters() {
        return this.baseProductParameters.map((item, index) => {
          return {
            ...item,
            basePosition: index + 1,
            tablePosition: index + 1
          }
        })
      },
      sortedSelectedColumns() {
        return this.selectedColumns.sort((a, b) => a.basePosition - b.basePosition)
      },
    },
    methods: {
      showRowSelected(value) {
        this.$store.dispatch("table/setShowRowsCounter", value);
        this.$refs.rowSelect.toggleDropdown();

        if (this.showRowsValue + this.showRowsCounter + 1 > this.tableDataLength) {
          this.$store.dispatch('table/setMaxValueReachedFlag', true)
        } else {
          this.$store.dispatch('table/setMaxValueReachedFlag', false)
        }
      },
      paginationBtnClicked(type) {
        this.$store.dispatch("table/deleteAllFromSelectedRows");

        if (type === 'prev') {
          if (this.showRowsValue - this.showRowsCounter > 0) {
            this.$store.dispatch("table/setShowRowsValue", this.showRowsValue - this.showRowsCounter);
          } else {
            this.$store.dispatch("table/setShowRowsValue", 0);
          }

          if (this.maxValueReachedFlag) {
            this.$store.dispatch('table/setMaxValueReachedFlag', false)
          }
        } else {
          if (this.showRowsValue + this.showRowsCounter * 2 + 1 <= this.tableDataLength) {
            console.log(this.showRowsValue, this.showRowsCounter, this.tableDataLength)
            this.$store.dispatch("table/setShowRowsValue", this.showRowsValue + this.showRowsCounter);

            if (this.maxValueReachedFlag) {
              this.$store.dispatch('table/setMaxValueReachedFlag', false)
            }
          } else {
            if (!this.maxValueReachedFlag) {
              this.$store.dispatch("table/setShowRowsValue", this.showRowsValue + this.showRowsCounter);
            }
            this.$store.dispatch('table/setMaxValueReachedFlag', true)
          }
        }
      },
      columnSelected(e, params) {
        if (params.value) {
          this.selectedColumns = this.selectedColumns.filter(x => x.value !== params.name);
        } else {
          this.selectedColumns.push(this.productParameters.find(x => x.value === params.name))
        }

        this.$refs.selectAllCheckbox.stateChecked = this.sortedSelectedColumns.length === this.productParameters.length;
      },
      allColumnSelected(e, params) {
        if (params.value) {
          this.$refs.showRowsCheckbox.forEach(item => {
            item.stateChecked = false
          });

          this.selectedColumns = [];
        } else {
          this.$refs.showRowsCheckbox.forEach(item => {
            item.stateChecked = true
          });

          this.selectedColumns = this.productParameters;
        }
      },
      getPartCheckboxState() {
        return this.sortedSelectedColumns.length !== this.productParameters.length &&
          this.sortedSelectedColumns.length !== 0;
      },
      setSortActiveCol(value) {
        if (this.sortActiveCol === value) {
          this.sortActiveCol = null;
        } else {
          this.sortActiveCol = value
        }
      }
    }
  }
</script>
<style lang="scss">
  .control-panel {
    $self: &;

    display: flex;
    padding: 16px 0;

    &__block {
      display: flex;
      align-items: center;
      height: max-content;

      &:not(:last-child) {
        margin-right: 20px;
      }

      &:first-child {
        margin-right: auto;
      }

      &_pagination {
        & #{$self}__text {
          margin: 0 8px;
        }
      }
    }

    &__title {
      font-weight: 600;
      color: $secondary-text-color;
      margin-right: 8px;
    }

    &__sorting {
      height: 32px;
      display: flex;
      align-items: center;

      & #{$self}__label {
        color: $primary-text-color;
        padding: 4px 8px;
        border-radius: 2px;
        cursor: pointer;
        transition: all $base-interval linear;
        user-select: none;

        &:not(:last-child) {
          margin-right: 3px;
        }

        &_active {
          background-color: $primary-button-bg;
          color: $light;
        }
      }
    }
  }
</style>
