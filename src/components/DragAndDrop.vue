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
                        <button class="widget-item-content__buttonDrag">Icon</button>
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
                    <button class="widget-block__buttonDrag">Icon</button>
                    <span v-if="item === null">Empty</span>
                    <span v-else>{{ item.label }}</span>
                </div>
            </div>

            <button @click="handleAddMoreRow">Add more row</button>
        </div>
    </div>
</template>

<script>
import { defineComponent, toRaw } from 'vue';

const createWidget = (key, label, size, position) => {
    return { key, label, size, position }
}

const preventSelect = (e) => {
    e.preventDefault();
    e.stopPropagation();
    e.cancelBubble = true;
    e.returnValue = false;

    return false
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
        }
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

            // moving from list
            if (this.movingFromList) {
                console.log('moving from list')
                // prevent drop oustide
                if (!widgetBlock) {
                    alert('drop outside')
                    return
                }

                const sourceData = JSON.parse(widgetStringified)
                const targetData = JSON.parse(widgetBlock.getAttribute('data'))

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
                })
            }

            // moving from grid
            else {
                console.log('moving from grid')
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
        handleAddMoreRow() {
            this.showMoreRow = true
        },

        // utilities
        __retrieveBlockData(item, index) {
            const key = item === null ? '' : item.key
            const size = item === null ? 1 : item.size
            const label = item === null ? 'Empty' : item.label

            return {
                key,
                size,
                label,
                position: index,
            }
        }
    },
    watch: {
        listWidgetBlock(newVal) {
            console.log('listWidgetBlock', newVal)
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

.grid-layout {
    flex: 1;
    /* gap: 10px; */
    display: flex;
    flex-wrap: wrap;
}

.widget-block {
    height: 120px;
    border: 1px solid black;
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

.widget-block__buttonDrag {
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