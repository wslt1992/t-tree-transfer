<template>
  <div class="home">
    <TurboTreeTransfer :personFromData = 'data'/>
  </div>
</template>

<script>
// @ is an alias to /src
import TurboTreeTransfer from '@/views/TurboTreeTransfer'
import data from '@/views/data'
export default {
  name: 'home',
  data () {
    return {
      data: JSON.parse(data).data.group
    }
  },
  components: {
    TurboTreeTransfer
  }
}
</script>
