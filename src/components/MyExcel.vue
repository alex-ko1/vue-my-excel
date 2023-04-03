<template>
  <div class="header">
    <div class="cellName">{{ headerLetter[colActive].name }}{{ rowActive + 1 }}</div>
    <div class="input">
      <input
        type="text"
        name=""
        id=""
        :value="arrCells[rowActive][colActive].content"
        @input="arrCells[rowActive][colActive].content = ($event.target as HTMLInputElement)!.value"
      />
    </div>
  </div>
  <div class="table-wrapper">
    <table class="table">
      <tr>
        <th></th>
        <th
          v-for="(col, i) in cols"
          :key="i"
          :class="{ 'col-active': i == colActive }"
          :style="{
            width: tHead[i]?.width + 'px',
            maxWidth: tHead[i]?.width + 'px',
          }"
        >
          {{ tHead[i]?.name }}
          <span class="col-resize" @mousedown="colResize(i, $event)"></span>
        </th>
      </tr>
      <tr v-for="(row, rowInd) in arrCells" :key="rowInd" ref="refRows">
        <td
          :class="{ 'row-active': rowInd == rowActive }"
          :style="{ height: rowsHeight[rowInd] + 'px' }"
        >
          {{ rowInd + 1 }}
          <span
            class="row-resize"
            @mousedown="rowResize(rowInd, $event)"
          ></span>
        </td>
        <td
          v-for="(col, colInd) in arrCells[rowInd]"
          :key="colInd"
          :class="{ active: col.active, selected: col.selected }"
          @click="activeCell(col, $event)"
          @keypress.enter.prevent="onNextCell(col)"
          @keyup.down.prevent="onNextCell(col)"
          @keyup.up.prevent="onUpCell(col)"
          @keyup.left.prevent="onLeftCell(col)"
          @keyup.right.prevent="onRightCell(col)"
        >
          <div
            class="cell-content"
            :contenteditable="col.editable"
            @input="col.content =($event.target as HTMLElement).textContent ?? '' "
          >
            {{ col.content }}
          </div>
          <span
            class="select-cells"
            @mousedown="selectCells"
            v-if="rowActive == rowInd && colActive == colInd"
          ></span>
        </td>
      </tr>
    </table>
  </div>
</template>

<script setup lang="ts">
import {computed, nextTick, ref} from "vue"

const props = defineProps({
  cols: {type: Number, default: 26},
  rows: {type: Number, default: 11}
})

interface Cell {
  content: string,
  active: boolean,
  selected: boolean,
  editable: boolean,
  width: number,
  row: number,
  col: number,
}

const arrCells = ref<Cell[][]>([]),
  tHead = ref<{ name: string, width: number }[]>([]),
  rowsHeight = ref<number[]>([]),
  colActive = ref<number>(0),
  rowActive = ref<number>(0);

const refRows = ref([]);
const onCreated = () => {
  for (let i = 0; i < props.rows; i++) {
    arrCells.value.push([]);
    rowsHeight.value.push(30);
    for (let j = 0; j < props.cols; j++) {
      arrCells.value[i].push({
        content: "",
        active: false,
        selected: false,
        editable: false,
        width: 100,
        row: i,
        col: j,
      });
    }
  }
  arrCells.value[0][0].active = true;
  arrCells.value[0][0].editable = true;
  const focusOnCell = async () => {
    await nextTick();
    (refRows.value[0] as any).children[1].firstChild.focus();
  };
  focusOnCell();
}
onCreated()

const headerLetter = computed(() => {
  const letterArr: string[] = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"];
  for (let i = 0; i < props.cols; i++) {
    if (i < 26) {
      tHead.value.push({name: letterArr[i], width: 100});
    } else {
      const colLetter =
        letterArr[Math.floor(i / 26) - 1] + letterArr[i % 26];
      tHead.value.push({name: colLetter, width: 100});
    }
  }
  return tHead.value;
})
const activeCell = (cell: Cell, e: Event) => {
  let cellActive;
  for (let i = 0; i < arrCells.value.length; i++) {
    cellActive = arrCells.value[i].find((el) => el.active == true);
    if (cellActive) {
      cellActive.active = false;
      cellActive.editable = false;
      break;
    }
  }
  colActive.value = cell.col;
  rowActive.value = cell.row;
  cell.active = true;
  cell.editable = true;
  const focusOnCell = async () => {
    await nextTick();
    (e.target as HTMLElement).focus();
  };
  focusOnCell()
}
const onNextCell = (cell: Cell) => {
  if (rowActive.value + 1 < props.rows) {
    cell.active = false;
    cell.editable = false;
    rowActive.value += 1;
    arrCells.value[rowActive.value][colActive.value].active = true;
    arrCells.value[rowActive.value][colActive.value].editable = true;
    const focusOnCell = async () => {
      await nextTick();
      (refRows.value[rowActive.value] as any).children[colActive.value + 1].firstChild.focus();
    };
    focusOnCell()
  }
}
const onKeypress = (cell: Cell) => {
  cell.active = false;
  cell.editable = false;
  arrCells.value[rowActive.value][colActive.value].active = true;
  arrCells.value[rowActive.value][colActive.value].editable = true;
  const focusOnCell = async () => {
    await nextTick();
    (refRows.value[rowActive.value] as any).children[colActive.value + 1].firstChild.focus();
  };
  focusOnCell()
}
const onUpCell = (cell: Cell) => {
  if (rowActive.value > 0) {
    rowActive.value -= 1;
    onKeypress(cell);
  }
}
const onLeftCell = (cell: Cell) => {
  if (colActive.value > 0) {
    colActive.value -= 1;
    onKeypress(cell);
  }
}
const onRightCell = (cell: Cell) => {
  if (colActive.value + 1 < props.cols) {
    colActive.value += 1;
    onKeypress(cell);
  }
}
const colResize = (i: number, e: Event) => {
  let target = e.target as HTMLElement;
  let mouseE = e as MouseEvent;

  const col = target.parentElement;
  const colWidth = col!.offsetWidth;
  const colX = mouseE.pageX;
  const mouseMove = (event: Event) => {
    let mouseEvent = event as MouseEvent;

    tHead.value[i].width = colWidth - (colX - mouseEvent.pageX);
  };
  document.addEventListener("mousemove", mouseMove);
  document.onmouseup = function () {
    document.removeEventListener("mousemove", mouseMove);
    col!.onmouseup = null;
  };
  col!.ondragstart = function () {
    return false;
  };
}
const rowResize = (i: number, e: Event) => {
  let target = e.target as HTMLElement;
  let mouseE = e as MouseEvent;

  const row = target.parentElement;
  const rowHeight = row!.offsetHeight;
  const rowY = mouseE.pageY;
  const mouseMove = (event: Event) => {
    let mouseEvent = event as MouseEvent;
    rowsHeight.value[i] = rowHeight - (rowY - mouseEvent.pageY);
    // row.style.maxWidth = rowWidth - (rowX - event.pageX) + "px";
  };
  document.addEventListener("mousemove", mouseMove);
  document.onmouseup = function () {
    document.removeEventListener("mousemove", mouseMove);
    row!.onmouseup = null;
  };
  row!.ondragstart = function () {
    return false;
  };
};
const selectCells = (e: Event) => {
  let mouseE = e as MouseEvent;

  let startSelectedX = mouseE.pageX;
  let startSelectedY = mouseE.pageY;
  const mouseMove = (event: Event) => {
    let mouseEvent = event as MouseEvent;

    let selectedCellsX = mouseEvent.pageX - startSelectedX;
    let selectedCellsY = mouseEvent.pageY - startSelectedY;
    let absSelectedCellsX = Math.abs(selectedCellsX);
    let absSelectedCellsY = Math.abs(selectedCellsY);
    let i = 1;
    const checkSelectedCells = () => {
      //   if (this.colActive + i + 1 == this.cols) {
      //     this.arrCells[this.rowActive][this.colActive + i].selected = true;
      //     selectedCellsX -= this.tHead[this.colActive + i].width;
      //     return;
      //   }
      if (absSelectedCellsX >= absSelectedCellsY) {
        arrCells.value.forEach((row) => {
          if (row[colActive.value].selected) {
            row[colActive.value].selected = false;
          }
        });
        if (colActive.value + i < props.cols && colActive.value + i > 0) {
          if (selectedCellsX < 0) {
            if (
              Math.abs(selectedCellsX) -
              tHead.value[colActive.value + i - 1].width >
              0
            ) {
              arrCells.value[rowActive.value][
              colActive.value + i - 1
                ].selected = true;
              selectedCellsX += tHead.value[colActive.value + i - 1].width;
              i--;
              checkSelectedCells();
            } else {
              arrCells.value[rowActive.value][
              colActive.value + i - 1
                ].selected = false;
              selectedCellsX -= tHead.value[colActive.value + i - 1].width;
              i++;
            }
          } else if (
            selectedCellsX - tHead.value[colActive.value + i].width >
            0
          ) {
            arrCells.value[rowActive.value][colActive.value + i].selected =
              true;
            selectedCellsX -= tHead.value[colActive.value + i].width;
            i++;
            checkSelectedCells();
          } else {
            arrCells.value[rowActive.value][colActive.value + i].selected =
              false;
            selectedCellsX += tHead.value[colActive.value + i].width;
            i--;
          }
        }
      } else {
        arrCells.value[rowActive.value].find((cell) => {
          cell.selected = false;
        });
        if (rowActive.value + i < props.rows && rowActive.value + i > 0) {
          if (selectedCellsY < 0) {
            if (
              Math.abs(selectedCellsY) >
              rowsHeight.value[rowActive.value + i - 1]
            ) {
              arrCells.value[rowActive.value + i - 1][
                colActive.value
                ].selected = true;
              selectedCellsY += rowsHeight.value[rowActive.value + i - 1];
              i--;
              checkSelectedCells();
            } else {
              arrCells.value[rowActive.value + i - 1][
                colActive.value
                ].selected = false;
              selectedCellsY -= rowsHeight.value[rowActive.value + i - 1];
              i++;
            }
          } else {
            if (selectedCellsY > rowsHeight.value[rowActive.value + i]) {
              arrCells.value[rowActive.value + i][colActive.value].selected =
                true;
              selectedCellsY -= rowsHeight.value[rowActive.value + i];
              i++;
              checkSelectedCells();
            } else {
              arrCells.value[rowActive.value + i][colActive.value].selected =
                false;
              selectedCellsY += rowsHeight.value[rowActive.value + i];
              i--;
            }
          }
        }
      }
    };
    checkSelectedCells();
  };
  document.addEventListener("mousemove", mouseMove);
  document.onmouseup = (event) => {
    document.removeEventListener("mousemove", mouseMove);
    document.onmouseup = null;
    // let selectedCells = event.pageX - startSelectedX;
    // let i = 1;
    arrCells.value[rowActive.value].find((cell) => {
      if (cell.selected) {
        cell.content =
          arrCells.value[rowActive.value][colActive.value].content;
        cell.selected = false;
      }
    });
    arrCells.value.forEach((row) => {
      if (row[colActive.value].selected) {
        row[colActive.value].content =
          arrCells.value[rowActive.value][colActive.value].content;
        row[colActive.value].selected = false;
      }
    });
  };
  // e.target.ondragstart = function () {
  //     return false;
  // };
};
</script>

<style lang="scss" scoped>
.header {
  display: flex;
  align-items: center;
  margin: 10px 0 10px 10px;
  align-items: stretch;
  .cellName {
    width: 5%;
    border: 1px solid #ccc;
    border-bottom-left-radius: 10px;
    border-top-left-radius: 10px;
    padding: 5px 10px;
    text-align: center;
  }

  .input {
    width: 96%;
    input {
      height: 100%;
      width: 98%;
      font-size: 1em;
      border-bottom-right-radius: 10px;
      border-top-right-radius: 10px;
      border: 1px solid #ccc;
      padding: 5px 10px;
      outline: none;
    }
  }
}
.table-wrapper {
  width: 98vw;
  height: 90vh;
  overflow: scroll;
  // border: 1px solid;
  margin: 10px;
  user-select: none;
}
table {
  width: max-content;
  border-collapse: collapse;
  cursor: cell;
  tr {
    td,
    th {
      border: 1px solid #ccc;
      height: 30px;
      text-align: center;
    }
    th {
      max-width: 100px;
      width: 100px;
      position: relative;
      cursor: s-resize;
      .col-resize {
        width: 7px;
        height: 100%;
        position: absolute;
        right: -4px;
        z-index: 1;
        cursor: col-resize;
      }
    }
    td {
      outline: none;
      position: relative;
      padding: 0;
      &.active {
        outline: 2px solid #008200;
      }
      &.selected {
        outline: 2px solid #007e00a5;
      }
      //   vertical-align: bottom;
      &:first-child {
        position: relative;
        cursor: e-resize;
        .row-resize {
          width: 100%;
          height: 6px;
          position: absolute;
          left: 0;
          bottom: -3px;
          z-index: 1;
          cursor: row-resize;
        }
      }
      .cell-content {
        outline: none;
        width: 100%;
        height: 100%;
      }
      .select-cells {
        position: absolute;
        z-index: 2;
        right: -5px;
        bottom: -5px;
        width: 7px;
        height: 7px;
        border: 1px solid #fff;
        background-color: #008200;
        cursor: crosshair;
      }
    }
  }
  .col-active {
    background-color: rgb(0, 0, 0, 0.1);
    border-bottom: 2px solid #008200;
  }
  .row-active {
    background-color: rgb(0, 0, 0, 0.1);
    border-right: 2px solid #008200;
  }
}
</style>
