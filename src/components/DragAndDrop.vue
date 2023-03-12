<template>
    <div class="container widget-settings" @mouseup="handleMouseUpContainer">
        <div class="list-area">
            <div class="search-bar">Searchbar</div>

            <div class="widget-list">
                <div v-for="item in listAvailableWidget" :draggable="true" :key="item.key"
                    @dragstart="handleDragFromList($event, item)" class="widget-item">
                    <div class="widget-item-content">
                        <span class="widget-item-content__label">{{ item.label }}</span>
                        <span class="widget-item-content__size">Size: {{ item.size }}</span>
                        <button class="widget-item-content__buttonDrag">Drag</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="grid-area">
            <div class="grid-layout dropzone" @drop="handleDrop($event)" @dragenter.prevent @dragover.prevent>
                <div v-for="(item, index) in listWidgetBlock" :key="index" class="widget-block" :class="[
                    item.key === 'empty' ? 'empty' : 'item',
                    `size-${item.size}`
                ]" :draggable="true" @dragstart="handleDragFromGrid($event, __retrieveBlockData(item, index))"
                    :data="JSON.stringify(__retrieveBlockData(item, index))">
                    <div class="widget-block-content">
                        <button class="widget-block__buttonDrag">Drag</button>
                        <button class="widget-block__buttonRemove" @click="handleRemoveBlock(index)">-</button>
                        <span v-if="item === null">Empty</span>
                        <span v-else>{{ item.label }}</span>
                    </div>
                </div>
            </div>

            <button @click="handleToggleRow" class="button-add-more">
                {{ !showMoreRow ? 'Add more row' : 'Hide row' }}
            </button>
        </div>
    </div>
</template>

<script>
import { defineComponent, toRaw } from 'vue';

const createWidget = (key, label, size, position) => {
    return { key, label, size, position }
}

const swapPosition = (arr, sourceIndex, targetIndex) => {
    const temp = arr[sourceIndex]
    arr[sourceIndex] = arr[targetIndex]
    arr[targetIndex] = temp
}

const DATA_TRANSFER_KEY = 'WIDGET'

export default defineComponent({
    name: 'DragAndDrop',
    data() {
        return {
            listAvailableWidget: [
                createWidget('widget-1', 'Widget 1', 1, -1),
                createWidget('widget-2', 'Widget 2', 2, -1),
                createWidget('widget-3', 'Widget 3', 2, -1),
                createWidget('widget-4', 'Widget 4', 1, -1),
                createWidget('widget-5', 'Widget 5', 4, -1),
                createWidget('widget-6', 'Widget 6', 3, -1),
                createWidget('widget-7', 'Widget 7', 1, -1),
                createWidget('widget-8', 'Widget 8', 1, -1),
                createWidget('widget-9', 'Widget 9', 1, -1),
                createWidget('widget-10', 'Widget 10', 1, -1),
                createWidget('widget-11', 'Widget 11', 1, -1),
            ],
            listActiveWidget: [

            ],
            listWidgetBlock: [],
            showMoreRow: false,
            movingFromList: false,
        }
    },
    computed: {
        slotCount() {
            if (this.showMoreRow) return 12
            return 8
        },
    },
    methods: {
        handleClick(event) {
            console.log(event.target)
        },
        handleDragFromList(event, item) {
            // console.log('@handleDragFromList', item)
            this.movingFromList = true
            event.dataTransfer.dropEffect = 'move'
            event.dataTransfer.effectAllowed = 'move'
            event.dataTransfer.setData(DATA_TRANSFER_KEY, JSON.stringify(item))
        },
        handleDragFromGrid(event, item) {
            this.movingFromList = false
            event.dataTransfer.dropEffect = 'move'
            event.dataTransfer.effectAllowed = 'move'
            event.dataTransfer.setData(DATA_TRANSFER_KEY, JSON.stringify(item))
            console.log('@handleDragFromGrid', item)
        },
        handleDrop(event) {
            const widgetStringified = event.dataTransfer.getData(DATA_TRANSFER_KEY)
            const widgetBlock = event.target.closest('.widget-block')

            if (!widgetBlock) {
                alert('drop outside')
                return
            }

            const sourceData = JSON.parse(widgetStringified)

            // moving from list
            if (this.movingFromList) {
                console.log('moving from list')

                // prevent double add
                if (this.listWidgetBlock.some(item => item?.key === sourceData.key)) {
                    alert(`item has already added`)
                    return
                }

                let offset = 0
                let row = 0
                let validSlotsLeftInRow = 0
                const firstAvailableSlotIndex = this.listWidgetBlock.findIndex(item => !item.key)

                const listSelectedWidget = this.listWidgetBlock.filter(item => item.key)

                for (let i = 0; i < listSelectedWidget.length; i++) {
                    const __block = listSelectedWidget[i];
                    offset += __block.size
                }

                if (offset % 4 === 0) row = (offset / 4) + 1 // move to next row when full
                else row = Math.ceil(offset / 4)

                validSlotsLeftInRow = (4 * row) - offset

                const isOverflow = sourceData.size > validSlotsLeftInRow || sourceData.size + offset > this.slotCount

                if (isOverflow) {
                    console.log(`item's size is unfit`)
                    return
                }

                this.listWidgetBlock.splice(firstAvailableSlotIndex, sourceData.size, {
                    ...sourceData,
                    position: firstAvailableSlotIndex,
                    size: sourceData.size
                }) // replace first empty slot with source widget

                this.listWidgetBlock = this.listWidgetBlock.map((o, i) => ({ ...o, position: i })) // re-indexing position

                console.log(toRaw(this.listWidgetBlock))
            }

            // moving from grid
            else {
                console.log('moving from grid')
                const targetData = JSON.parse(widgetBlock.getAttribute('data'))

                if (targetData.size !== sourceData.size) {
                    alert(`item's size must be the same size`)
                    return
                }

                if (!targetData.key) {
                    // when swap with empty slot
                    /** find the first available slot and replace it with the source widget */
                    const firstAvailableSlotIndex = this.listWidgetBlock.findIndex(item => !item.key)
                    swapPosition(this.listWidgetBlock, firstAvailableSlotIndex, sourceData.position)
                    this.listWidgetBlock.splice(sourceData.position, 1)
                    this.listWidgetBlock.push(createWidget('', 'Empty', 1, this.listWidgetBlock.length))
                    console.log(this.listWidgetBlock)
                    return
                }


                swapPosition(this.listWidgetBlock, targetData.position, sourceData.position)
            }
        },

        // prevent
        handleMouseDownContainer(event) {
            console.log('@mousedown', event)
            event.preventDefault()
        },

        handleMouseUpContainer(event) {

        },
        handleDragOnIcon(event) {
            event.stopPropagation()
            console.log('@dragOnIcon', event)
        },
        handleClickDragIcon(event) {
            console.log(event)
        },
        handleMouseDownDragIcon(event) {
            console.log('@handleMouseDownDragIcon', event)
            event.stopPropagation()
        },
        handleRemoveBlock(index) {
            const target = this.listWidgetBlock[index]
            this.listWidgetBlock.splice(index, 1)
            console.log('target.size', target.size)
            for (let i = 0; i < target.size; i++) {
                this.listWidgetBlock.push(createWidget('', 'Empty', 1, target.position + i))
            }
        },
        handleToggleRow() {
            this.showMoreRow = !this.showMoreRow
        },

        // utilities
        __retrieveBlockData(item, index) {
            const key = item === null ? '' : item.key
            const size = item === null ? 1 : item.size
            const label = item === null ? 'Empty' : item.label

            return { key, size, label, position: index }
        }
    },
    watch: {
        showMoreRow(newVal) {
            if (newVal) {
                console.log('showMoreRow')
                const lastEl = this.listWidgetBlock.slice(-1)
                for (let i = 1; i <= 4; i++) {
                    this.listWidgetBlock.push(createWidget('', 'Empty', 1, lastEl + i))
                }
            }
            else {
                // this.listWidgetBlock
            }
        }
    },
    created() {
        // set default for block grid
        for (let index = 0; index < this.slotCount; index++) {
            this.listWidgetBlock.push({
                key: '',
                label: 'Empty',
                position: index,
                size: 1,
            })
        }
    }
})
</script>

<style scoped>
/* grid gap is 10px */
.widget-settings {
    --grid-gap: 10px;
}

/* container */
.container {
    display: flex;
    width: 100%;
    gap: 10px;
}

/* widget list */
.list-area {
    flex: 1
}

.widget-list {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.widget-item {
    border: 1px solid black;
    position: relative;
    height: 30px;
    display: flex;
    align-items: center;
    padding: 0.25rem;
}

.widget-item-content {
    display: flex;
    justify-content: space-between;
    width: 100%;
}

.widget-item-content__label {}

.widget-item-content__size {
    margin-right: 50px;
}

.widget-item-content__buttonDrag {
    position: absolute;
    top: 3px;
    right: 3px;
}

/* widget grid */
.grid-area {
    flex: 1;
}

.grid-area .button-add-more {
    margin-top: 1rem;
}

.grid-layout {
    flex: 1;
    /* gap: 10px; */
    display: flex;
    flex-wrap: wrap;
}

.widget-block {
    height: 120px;
    position: relative;
    flex-shrink: 0;
    flex-grow: 0;
}

.widget-block.size-1 {
    flex-basis: 25%;
}

.widget-block.size-2 {
    flex-basis: 50%;
}

.widget-block.size-3 {
    flex-basis: 75%;
}

.widget-block.size-4 {
    flex-basis: 100%;
}

.widget-block.empty {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-basis: 1;
}

.widget-block.item {
    display: flex;
    align-items: center;
    justify-content: center;
}


.widget-block-content {
    border: 1px solid black;
    width: calc(100% - 5px);
    height: calc(100% - 5px);
    display: flex;
    justify-content: center;
    align-items: center;
}


.widget-block__buttonDrag {
    position: absolute;
    right: 2rem;
    top: 0.5rem
}

.widget-block__buttonRemove {
    position: absolute;
    right: 0.5rem;
    top: 0.5rem
}

.prevent-select {
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
</style>