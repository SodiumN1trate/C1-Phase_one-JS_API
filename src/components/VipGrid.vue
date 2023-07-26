<template>
  <div>
    <div v-if="!fullscreen">
      <h1>Upload Vip's</h1>
      <section
        class="file-drop"
        ref="file-drop"
        @drop="fileDrop($event)"
        @dragenter="fileDragEnter($event)"
        @dragleave="fileDragLeave($event)"
        @dragover="$event.preventDefault()"
      >
        Drag file
      </section>
    </div>
    <br>
    <br>
    <div v-if="!fullscreen">
      <h1>Vip's sample</h1>
      <button @click="loadSample">Load sample</button>
      <br>
      <br>
    </div>
    <h1>Full screen</h1>
    <button @click="fullScreen">{{ fullscreen ? 'Exit' : 'Enter' }} full-screen</button>
    <br>
    <br>
    <h1>Vip's grid</h1>
    <div class="grid">
      <div
        v-for="(box, index) in grid"
        :key="index"
        draggable="true"
        class="grid-box"
        @drag="drag($event, index)"
        @drop="drop($event, index)"
        @dragenter="$event.preventDefault()"
        @dragover="$event.preventDefault()"
      >
          {{ box }}
      </div>
    </div>
    <div class="export"  v-if="!fullscreen">
      <br>
      <br>
      <h1>Export VIP list</h1>
      <div>
        <button @click="copy" style="align-self: start">Copy-to-clipboard</button>
        <button @click="exportPlain()" style="align-self: start">Plain text file</button>
      </div>
      <br>
      <textarea v-model="exportMarkdown" rows="10"></textarea>
      <br>
      <br>
      <a download="vip-list.txt" ref="download"></a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'VipGrid',
  data () {
    return {
      grid: [],
      dragging: null,
      fullscreen: null,
      exportMarkdown: null
    }
  },
  created () {
    this.grid = JSON.parse(localStorage.getItem('grid')) || []
    this.render()
  },
  methods: {
    exportPlain () {
      const fileReader = new FileReader()
      const file = new File([this.exportMarkdown], "foo.txt", {
        type: "text/plain",
      });
      fileReader.onload = (e) => {
        this.$refs.download.href = fileReader.result
        this.$refs.download.click()
      }
      fileReader.readAsDataURL(file)
    },
    copy () {
      navigator.clipboard.writeText(this.exportMarkdown)
      alert('Copied!')
    },
    fullScreen () {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen()
        this.fullscreen = true
      } else {
        document.exitFullscreen()
        this.fullscreen = false
      }
      console.log(document.fullscreenElement)
    },
    fileDragLeave (e) {
      e.preventDefault()
      this.$refs["file-drop"].style.border = '2px dashed #e8e8e8'
    },
    fileDragEnter (e) {
      e.preventDefault()
      this.$refs["file-drop"].style.border = '2px dashed dodgerblue'
    },
    fileDrop (e) {
      e.preventDefault()
      this.$refs["file-drop"].style.border = '2px dashed #e8e8e8'
      const fileReader = new FileReader()
      fileReader.onload = (evt) => {
        let text = evt.target.result.split('\n')
        text.shift()
        text.shift()
        this.grid = []
        text.forEach((row) => {
          if (row[0] === '-') {
            this.grid.push(row.slice(2))
          }
        })
        this.render()
      }
      fileReader.readAsText(e.dataTransfer.files[0])
    },
    saveGrid () {
      localStorage.setItem('grid', JSON.stringify(this.grid))
    },
    loadSample () {
      this.grid = [ 'Andrew', 'Robert', 'Steve']
      this.render()
    },
    drop (event, index) {
      let temp = this.grid[index]
      this.grid[index] = this.grid[this.dragging]
      this.grid[this.dragging] = temp
      this.render()
    },
    drag (event, index) {
      this.dragging = index
    },
    render () {
      this.exportMarkdown = "#VIP List\n\n"
      let temp = []
      for(let i = 0; i < 32; i++) {
        if(this.grid[i]) {
          temp.push(this.grid[i])
          this.exportMarkdown += '- ' + this.grid[i] + '\n'
        } else {
          temp.push('')
          this.exportMarkdown += '- \n'
        }
      }
      this.grid = temp
      this.saveGrid()
    }
  }
}
</script>

<style scoped>
.grid {
    display: grid;
    grid-template-columns: repeat(8, 100px);
    grid-template-rows: repeat(4, 1fr);
    gap: 20px;
}
.grid-box {
    border: 1px solid #e8e8e8;
    text-align: center;
    padding: 16px;
}
.file-drop {
    border: 2px dashed #e8e8e8;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 200px;
    transition: .3s all;
}

.export {
    display: flex;
    align-items: stretch;
    justify-content: start;
    flex-direction: column;
}

.export > div {
    display: flex;
    align-self: start;
    justify-content: space-between;
    gap: 20px;
}
</style>
