<template>
  <modal v-if=show @close="hideUploadPopup()">
    <div slot="body">
      <div id="dropTarget"><span id="dropText" v-html="$t('fileDrop.dropKml')"></span></div>
    </div>
  </modal>
</template>

<script>
import Modal from './Modal'

export default {
  components: {
    Modal
  },
  props: ['show'],
  methods: {
    hideUploadPopup() {
      this.$emit('disable')
    }
  },
  watch: {
    show(show) {
      let counter = 0

      const dragEnter = function(e) {
        e.stopPropagation()
        e.preventDefault()
        counter++
        document.getElementById('dropTarget').classList.add('dragover')
      }

      const dragLeave = function(e) {
        counter--
        if (counter === 0) {
          document.getElementById('dropTarget').classList.remove('dragover')
        }
      }

      const dragOver = function(e) {
        e.stopPropagation()
        e.preventDefault()
      }

      const handleFiles = files => {
        const file = files[0] // only the first file is shown - TODO show error
        // TODO check file type
        // if (file.type !== 'application/xml') {
        //   alert('Not a KML file')
        //   return
        // }

        const reader = new FileReader()
        reader.onload = e => {
          try {
            this.$store.commit('overlay_kml', { kml: e.target.result })
          } catch (err) {
            alert('Error reading the KML file:\n' + err)
          }
          this.hideUploadPopup()
        }
        reader.readAsText(file)
      }

      const drop = function(e) {
        e.stopPropagation()
        e.preventDefault()

        const dt = e.dataTransfer,
              files = dt.files

        handleFiles(files)

        counter = 0
      }
      if (show) {
        this.$nextTick(() => {
          const dropTarget = document.getElementById('dropTarget')
          dropTarget.addEventListener('dragenter', dragEnter, false)
          dropTarget.addEventListener('dragleave', dragLeave, false)
          dropTarget.addEventListener('dragover', dragOver, false)
          dropTarget.addEventListener('drop', drop, false)
        })
      }
    }
  }
}
</script>

<style>
#dropTarget {
  width: 300px;
  height: 186px;
  line-height: 186px;
  border: 3px dashed grey;
  font-weight: bold;
  text-align: center;
  font-size: 32px;
  color: #999;
}
#dropTarget.dragover {
  border-color: #319FD3;
}
#dropText {
  display: inline-block;
  vertical-align: middle;
  line-height: normal;
}
</style>
