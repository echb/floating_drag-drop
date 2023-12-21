<script setup lang="ts" >
import { ComputedRef, Ref, computed, ref, toRaw } from 'vue';
type boxType = 'textArea' | 'sqr'
type item = { x: number, y: number, type: boxType, w: number, h: number, text?: string, color?: string }
class Item {
  id: number;
  x: number;
  y: number;
  w: number;
  h: number;
  type: string;
  text: string | undefined;
  color: string | undefined;

  constructor(item: item) {
    this.id = Math.random()
    this.x = item.x
    this.y = item.x
    this.w = item.w
    this.h = item.h
    this.type = item.type
    this.text = item.text
    if (item.text === undefined) {
      this.color = item?.color ?? '#ff0000'
    }
  }
}
const selected: Ref<Item> = ref(new Item({ x: 10, y: 100, type: 'textArea', w: 100, h: 100 }))

let items = ref([
  new Item({ x: 150, y: 100, type: 'textArea', w: 100, h: 100, text: 'some text' }),
  new Item({ x: 350, y: 100, type: 'sqr', w: 100, h: 100, color: '#0000ff' }),
  new Item({ x: 200, y: 100, type: 'sqr', w: 100, h: 100 }),
])

let x: number
let y: number
let index: number
let observerSelectedItem: ResizeObserver

function onMouseDown(e: any, item: Item, i: number) {
  index = i
  selected.value = item
  x = e.layerX
  y = e.layerY
  observerSelectedItem.observe(document.querySelector(`[data-index="${index}"]`)!)
}
function onMouseUp(index: number) {
  observerSelectedItem.unobserve(document.querySelector(`[data-index="${index}"]`)!)
}

observerSelectedItem = new ResizeObserver(entries => {
  selected.value.w = Number(entries[0].target.style.width.replace('px', ''))
  selected.value.h = Number(entries[0].target.style.height.replace('px', ''))
})

function dropZone(ev: MouseEvent) {
  selected.value.x = ev.clientX - x
  selected.value.y = ev.clientY - y
  ev.preventDefault();
}

function end() {
  items.value = items.value.filter((e) => e.id !== selected.value.id)
  items.value.push(structuredClone(toRaw(selected.value)))
}

function add(type: boxType) {
  if (type === 'sqr') {
    items.value.push(new Item({ x: 200, y: 100, type: 'sqr', w: 100, h: 100, color: '#ff0000' }))
  }
  if (type === 'textArea') {
    items.value.push(new Item({ x: 200, y: 100, type: 'textArea', w: 100, h: 100 }))
  }
}

const reversedItems: ComputedRef<Array<Item>> = computed(() => {
  return items.value.toReversed()
})
async function copyToClipboard(text: string) {
  await navigator.clipboard.writeText(text)
}
function remove(item: Item) {
  items.value = items.value.filter((e) => e.id !== item.id)
}
</script>

<template>
  <div class="left_panel">
    <div class="selected">
      <textarea v-if="selected.type === 'textArea'" name="" id="" cols="30" rows="10" v-model="selected.text"></textarea>
      <div v-else-if="selected.type === 'sqr'" class="block">asds</div>
    </div>
    <div class="left_panel-mid">
      <button @click="add('sqr')">add box</button>
      <button @click="add('textArea')">add textbox</button>
      <div v-for="item in reversedItems" class="flex ml-1">
        <span class="delete mr-1 pointer" @click="remove(item)">X</span>
        <details>
          <summary>
            {{ item.type }}
          </summary>

          <div v-if="item.color">
            <label for="head">Color: </label>
            <input type="color" id="head" name="head" v-model="item.color" />
          </div>

          <div class="left_panel-box_text ml-4" v-if="item.text">
            <span>Text:</span> <span title="copy text" @click="copyToClipboard(item.text)">{{ item.text }}</span>
          </div>
        </details>
      </div>
    </div>
  </div>

  <div @dragover="dropZone" class="main_box" @dragend="end">
    <div v-for="(item, i) in items" draggable="true" class="floating_box" :style="`left: ${item.x}px;top: ${item.y}px;`"
      @mousedown="onMouseDown($event, item, i)" @mouseup="onMouseUp(i)">

      <textarea v-if="item.type === 'textArea'" name="" cols="30" rows="10"
        :style="`width: ${item.w}px; height: ${item.h}px;`" :data-index="i" v-model="item.text"></textarea>
      <div v-if="item.type === 'sqr'" :data-index="i" class="block"
        :style="`width: ${item.w}px; height: ${item.h}px; background-color:${item.color}`">
        {{ item.text }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.pointer {
  cursor: pointer;
}

.flex {
  display: flex;
}

.ma-1 {
  margin: 5px;
}

.mx-1 {
  margin-left: 5px;
  margin-right: 5px;
}

.ml-1 {
  margin-left: 5px;
}

.ml-4 {
  margin-left: 25px;
}

.mr-1 {
  margin-right: 5px;
}

/* -------------- */
ul {
  margin: 0px;
  padding: 0px;
}

/* -------------- */
.left_panel {
  max-width: 300px;
  overflow: scroll;
  display: inline-block;
}

.left_panel-mid {
  margin-top: 5px;
}

.left_panel-box_text {
  cursor: copy;
  max-width: 200px;
  text-overflow: ellipsis;
  overflow: hidden;
  -webkit-line-clamp: 1;
  line-clamp: 1;
  -webkit-box-orient: vertical;
  display: -webkit-box;
}

.delete {
  color: red;
}

.selected {
  padding: 20px;
  display: inline-block;
  background-color: rgb(53, 53, 53);
}

.main_box {
  width: 100%;
  height: 100%;
}

.floating_box {
  position: absolute;
}

.block {
  background-color: red;
  position: absolute;
  resize: both;
  overflow: scroll;
  border: 1px solid black;
}

.selected * {
  top: initial;
  left: initial;
  display: initial;
  position: initial;
  display: inline-block;
  width: 100px;
  height: 100px;
  margin: 0;
  padding: 0;
  overflow: hidden;
  resize: none;
}
</style>
