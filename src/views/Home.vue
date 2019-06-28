<template>
  <div class="home">
    <TurboTreeTransfer :fromData = 'data' @changeCheckedKeys="changeCheckedKeys"/>
    <br/>
    <div>{{keys}}</div>
  </div>
</template>

<script>
// @ is an alias to /src
// import TurboTreeTransfer from '@/views/TurboTreeTransfer'
import TurboTreeTransfer from 't-tree-transfer'
import data from '@/views/data'
export default {
  name: 'home',
  data () {
    return {
      data: JSON.parse(data).data.group,
      keys: []
    }
  },
  components: {
    TurboTreeTransfer
  },
  methods: {
    changeCheckedKeys (keys) {
      this.keys = keys
    }
  }
}
</script>
