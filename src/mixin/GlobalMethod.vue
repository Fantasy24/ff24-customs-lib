<template>

</template>

<script>
import {numberRule, requiredRule, specialCharRule} from 'ff24-js/src/utils'
import {preventChar} from "ff24-js/src/utils/ECustomsUtils";

export default {
  name: "GlobalMethod",
  methods: {
    requiredRule,
    numberRule,
    specialCharRule,
    preventChar
  }
}
</script>
