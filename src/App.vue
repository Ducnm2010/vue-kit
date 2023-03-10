<template>
  <div class="app">
    <div class="row">
      <div class="drop-zone" @drop="onDrop($event, 1)" @dragenter.prevent @dragover.prevent>
        <div class="drag-el" v-for="item in listOne" :key="item.title" draggable @dragstart="startDrag($event, item)">
          <span>{{ item.title }}</span>
        </div>
      </div>
      <div class="drop-zone" @drop="onDrop($event, 2)" @dragenter.prevent @dragover.prevent>
        <div class="drag-el" v-for="item in listTwo" :key="item.title" draggable @dragstart="startDrag($event, item)">
          <span>{{ item.title }}</span>
        </div>
      </div>
    </div>

    <div class="row">
      <DragAndDrop></DragAndDrop>
      <!-- <DragAndDrop2></DragAndDrop2> -->
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import DragAndDrop from './components/DragAndDrop.vue';
// import DragAndDrop2 from './components/DragAndDrop2.vue';

export default defineComponent({
  name: 'App',
  components: {
    DragAndDrop,
    // DragAndDrop2
  },
  data() {
    return {
      items: [
        {
          id: 0,
          title: 'Item A',
          list: 1,
        },
        {
          id: 1,
          title: 'Item B',
          list: 1,
        },
        {
          id: 2,
          title: 'Item C',
          list: 2,
        },
      ],
    }
  },
  computed: {
    listOne() {
      return this.items.filter((item) => item.list === 1)
    },
    listTwo() {
      return this.items.filter((item) => item.list === 2)
    },
  },
  methods: {
    startDrag(evt, item) {
      evt.dataTransfer.dropEffect = 'move'
      evt.dataTransfer.effectAllowed = 'move'
      evt.dataTransfer.setData('itemID', item.id)
    },
    onDrop(evt, list) {
      const itemID = evt.dataTransfer.getData('itemID')
      const item = this.items.find((item) => item.id == itemID)
      item.list = list
    },
  },
})
</script>

<style scoped>
.row {
  display: flex;
  width: 100%;
  gap: 10px;
}

.drop-zone {
  background-color: #eee;
  margin-bottom: 10px;
  padding: 10px;
  flex: 1;
}

.drag-el {
  background-color: #fff;
  margin-bottom: 10px;
  padding: 5px;
  pointer-events: fill;
}
</style>