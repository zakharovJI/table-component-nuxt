<template>
  <div
    class="brand-dropdown"
    :class="{ 'brand-dropdown_active': isDropdownActive }"
    v-on-clickaway="hideDropdown"
  >
    <div
      class="brand-dropdown__control"
      aria-haspopup="true"
      aria-controls="{'brand-dropdown__dropdown-menu'}"
      @click="toggleDropdown"
    >
      <div class="brand-dropdown__label"> {{ label }}</div>
      <svg-icon name="chevron-down" class="brand-dropdown__icon">
      </svg-icon>
    </div>
    <div
      class="brand-dropdown__dropdown-menu"
      id="dropdown-menu"
      role="menu"
    >
      <slot name="dropdown-list"/>
    </div>
  </div>
</template>
<script>
  import {mixin as clickaway} from 'vue-clickaway';

  export default {
    name: 'BrandDropdown',
    mixins: [clickaway],
    props: {
      label: {
        type: String,
        required: true,
      },
      value: {}
    },
    data() {
      return {
        isDropdownActive: false,
      }
    },
    methods: {
      hideDropdown() {
        this.isDropdownActive = false;
      },
      toggleDropdown() {
        this.isDropdownActive = !this.isDropdownActive;
      }
    }
  }
</script>
<style lang="scss">
  .brand-dropdown {
    $self: &;

    border: 1px solid $primary-border-color;
    border-radius: 2px;
    min-height: 32px;
    display: flex;
    width: max-content;
    cursor: pointer;
    position: relative;
    z-index: 100;

    &__control {
      display: flex;
      justify-content: center;
      align-items: center;
      color: $additional-text-color;
      margin: auto;
      width: 100%;
      padding: 0 16px;

    }

    &__label {
      color: inherit;
    }

    &__icon {
      color: inherit;
      width: 8px;
      height: 5px;
      margin-left: 7px;
    }

    &__dropdown-menu {
      position: absolute;
      display: none;
      border-radius: 4px;
      border: 1px solid $primary-border-color;
      padding: 9px 0;
      overflow: auto;
      max-height: 240px;
      background-color: $light;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.16);
      top: calc(100% + 14px);
      left: -1px;
      min-width: calc(100% + 2px);
    }

    &__dropdown-item {
      padding: 6px 17px;
      background-color: $light;
      cursor: pointer;
      white-space: nowrap;

      &:hover {
        background-color: darken($light, 5%);
      }

      &:active {
        background-color: darken($light, 10%);
      }

      &:focus {
        border-color: darken($primary-border-color, 5%);
      }

      &_no-padding {
        padding: 0;
      }

      & #{$self}__checkbox {
        width: 100%;

        label {
          width: 100%;
          padding: 6px 17px;
        }
      }

      &_all {
        & #{$self}__checkbox {
          label {
            font-weight: bold;
          }
        }
      }
    }

    &_active {
      & #{$self}__dropdown-menu {
        display: block;
      }
    }
  }
</style>
