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
                    :class="{ active: col.active, selected: col.selected,selectionDark : isSelected(rowInd,colInd) }"
                    @mousedown="activeCell(col, $event)"
                    @mouseover="isStartOver && onMouseOver(rowInd,colInd,$event)"
                    @mouseup="onMouseUp"
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
                        @mousedown="duplicateContent"
                        v-if="rowActive == rowInd && colActive == colInd && !duplicateSelection"
                    ></span>
                    <span class="duplicate-selection"
                          v-if="duplicateSelection && rowInd == bottomRightCell[0] && colInd == bottomRightCell[1]"
                          @mousedown="duplicateSelectCells"
                          @mouseover="duplicateSelectMove && onDuplicateSelectMove($event)"
                          @mouseup="duplicateSelectMove = false"
                    ></span>
                </td>
            </tr>
        </table>
    </div>
</template>

<script setup lang="ts">
import {computed, nextTick, reactive, ref} from "vue"

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

interface Selection {
    startRow: number | null,
    startCol: number | null,
    endRow: number | null,
    endCol: number | null,
}


const arrCells = ref<Cell[][]>([]),
    tHead = ref<{ name: string, width: number }[]>([]),
    rowsHeight = ref<number[]>([]),
    colActive = ref<number>(0),
    rowActive = ref<number>(0);

const refRows = ref([]);

let isStartOver = ref<boolean>(false);
let bottomRightCell: number[] = [0, 0];

const selection = reactive<Selection>({
    startRow: 0,
    startCol: 0,
    endRow: 0,
    endCol: 0,
});

let startRow: number, startCol: number, endRow: number, endCol: number;
const defaultSelectValue = () => {
    selection.startRow = null;
    selection.startCol = null;
    selection.endRow = null;
    selection.endCol = null;
}
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
const duplicateSelection = computed(()=>{
    if (selection.startRow == selection.endRow && selection.startCol == selection.endCol) {
        return false
    } else return true;
    // return rowActive.value !== selection.endRow || colActive.value !== selection.endCol
});
const isDuplicateSquare = computed(()=>{
   return rowActive.value == selection.endRow && colActive.value == selection.endCol
});
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

    selection.startRow = cell.row;
    selection.endRow = cell.row;
    selection.startCol = cell.col;
    selection.endCol = cell.col;
    isStartOver.value = true;
    bottomRightCell=[cell.row,cell.col]
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
        defaultSelectValue();
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
    focusOnCell();
    defaultSelectValue();
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
const duplicateContent = (e: Event) => {
    e.stopPropagation();
    let mouseE = e as MouseEvent;

    let startSelectedX = mouseE.pageX;
    let startSelectedY = mouseE.pageY;
    const mouseMove = (event: Event) => {
        isStartOver.value = false

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
                // Make the default value
                arrCells.value.forEach((row) => {
                    if (row[colActive.value].selected) {
                        row[colActive.value].selected = false;
                    }
                });

                if (colActive.value + i < props.cols && colActive.value + i > 0) {
                    if (selectedCellsX < 0) {
                        if (Math.abs(selectedCellsX) - tHead.value[colActive.value + i - 1].width > 0) {
                            arrCells.value[rowActive.value][colActive.value + i - 1].selected = true;
                            selectedCellsX += tHead.value[colActive.value + i - 1].width;
                            i--;
                            checkSelectedCells();
                        } else {

                            arrCells.value[rowActive.value][colActive.value + i - 1].selected = false;
                            selectedCellsX -= tHead.value[colActive.value + i - 1].width;
                            i++;
                        }
                    } else if (selectedCellsX - tHead.value[colActive.value + i].width > 0) {
                        arrCells.value[rowActive.value][colActive.value + i].selected = true;
                        selectedCellsX -= tHead.value[colActive.value + i].width;
                        i++;
                        checkSelectedCells();
                    } else {
                        arrCells.value[rowActive.value][colActive.value + i].selected = false;
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
                        if (Math.abs(selectedCellsY) > rowsHeight.value[rowActive.value + i - 1]) {
                            arrCells.value[rowActive.value + i - 1][colActive.value].selected = true;
                            selectedCellsY += rowsHeight.value[rowActive.value + i - 1];
                            i--;
                            checkSelectedCells();
                        } else {
                            arrCells.value[rowActive.value + i - 1][colActive.value].selected = false;
                            selectedCellsY -= rowsHeight.value[rowActive.value + i - 1];
                            i++;
                        }
                    } else {
                        if (selectedCellsY > rowsHeight.value[rowActive.value + i]) {
                            arrCells.value[rowActive.value + i][colActive.value].selected = true;
                            selectedCellsY -= rowsHeight.value[rowActive.value + i];
                            i++;
                            checkSelectedCells();
                        } else {
                            arrCells.value[rowActive.value + i][colActive.value].selected = false;
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
    document.onmouseup = () => {
        document.removeEventListener("mousemove", mouseMove);
        document.onmouseup = null;
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
const selectCells = (rowInd: number, colInd: number) => {
    selection.startRow = rowInd;
    selection.endRow = rowInd;
    selection.startCol = colInd;
    selection.endCol = colInd;
}
const onMouseOver = (rowInd: number, colInd: number, event: Event) => {
    if (selection.startRow === null || selection.startCol===null) {
        return
    }
    selection.endRow = rowInd;
    selection.endCol = colInd;
    bottomRightCell = [rowInd,colInd];
    if(selection.endCol < selection.startCol){
        bottomRightCell[1]= selection.startCol
    }
    if (selection.endRow < selection.startRow){
        bottomRightCell[0]= selection.startRow
    }
}
let selectedCellsValue:string[][] = [];
const onMouseUp = () => {
    isStartOver.value = false;
    selectedCellsValue = [];
    let iteration = 0;
    for (let row = startRow; row <= endRow; row++) {
            selectedCellsValue[iteration] = [];
        for (let col = startCol; col <= endCol; col++) {
            selectedCellsValue[iteration].push(arrCells.value[row][col].content)
        }
        iteration++;
    }
    // console.log(selectedCellsValue)
}
const isSelected = (rowInd: number, colInd: number) => {
    if (selection.startRow === null || selection.endRow === null ||
        selection.startCol === null || selection.endCol === null) {
        return false;
    }
    startRow = Math.min(selection.startRow, selection.endRow);
    endRow = Math.max(selection.startRow, selection.endRow);
    startCol = Math.min(selection.startCol, selection.endCol);
    endCol = Math.max(selection.startCol, selection.endCol);
    return (
        rowInd >= startRow &&
        rowInd <= endRow &&
        colInd >= startCol &&
        colInd <= endCol
    );
}

let duplicateSelectMove = false;

const duplicateSelectCells = (e:Event) =>{
    e.stopPropagation();
    duplicateSelectMove = true;

    let mouseE = e as MouseEvent;
    let startSelectedX = mouseE.pageX;
    let startSelectedY = mouseE.pageY;
    const mouseMove = (event: Event) => {
        isStartOver.value = false

        let mouseEvent = event as MouseEvent;
        let selectedCellsX = mouseEvent.pageX - startSelectedX;
        let selectedCellsY = mouseEvent.pageY - startSelectedY;
        let absSelectedCellsX = Math.abs(selectedCellsX);
        let absSelectedCellsY = Math.abs(selectedCellsY);
        let i = 1;

        const checkSelectedCells = () => {
            // Cursor horizontal movement
            if (absSelectedCellsX >= absSelectedCellsY) {
                arrCells.value.forEach((row) => {
                    for (let col = startCol; col <= endCol; col++){
                        if (row[col].selected) {
                            row[col].selected = false;
                        }
                    }
                });
                // If move to the right
                if (bottomRightCell[1] + i < props.cols && bottomRightCell[1] + i > 0){

                    if (selectedCellsX > 0) {
                        if (selectedCellsX - tHead.value[bottomRightCell[1] + i].width > 0) {
                            for (let row = startRow; row <= endRow; row++) {
                                arrCells.value[row][bottomRightCell[1] + i].selected = true;
                            }
                            selectedCellsX -= tHead.value[bottomRightCell[1] + i].width;
                            i++;
                            checkSelectedCells();
                        } else {
                            for (let row = startRow; row <= endRow; row++) {
                                arrCells.value[row][bottomRightCell[1] + i].selected = false;
                            }
                            selectedCellsX += tHead.value[bottomRightCell[1] + i].width;
                            i--;
                        }
                    } // If move to the left
                    else {
                        if (Math.abs(selectedCellsX) - tHead.value[bottomRightCell[1] + i - 1].width > 0) {
                            for (let row = startRow; row <= endRow; row++) {
                                arrCells.value[row][bottomRightCell[1] + i - 1].selected = true;
                            }
                            selectedCellsX += tHead.value[bottomRightCell[1] + i].width;
                            i--;
                            checkSelectedCells();
                        } else {
                            for (let row = startRow; row <= endRow; row++) {
                                arrCells.value[row][bottomRightCell[1] + i - 1].selected = false;
                            }
                            selectedCellsX -= tHead.value[bottomRightCell[1] + i].width;
                            i++;
                        }
                    }
                }
                // Cursor vertical movement
            } else {
                    for (let row = startRow; row <= endRow; row++){
                       arrCells.value[row].find((cell)=>{
                           cell.selected = false;
                       })
                    }
                if (bottomRightCell[0] + i < props.rows && bottomRightCell[0] + i > 0) {
                    // If move to the up
                    if (selectedCellsY < 0) {
                        if (Math.abs(selectedCellsY) > rowsHeight.value[bottomRightCell[0] + i - 1]) {
                            for (let col = startCol; col <= endCol; col++) {
                                arrCells.value[bottomRightCell[0] + i - 1][col].selected = true;
                            }
                            selectedCellsY += rowsHeight.value[bottomRightCell[0] + i - 1];
                            i--;
                            checkSelectedCells();
                        } else {
                            for (let col = startCol; col <= endCol; col++) {
                                arrCells.value[bottomRightCell[0] + i - 1][col].selected = false;
                            }
                            selectedCellsY -= rowsHeight.value[bottomRightCell[0] + i - 1];
                            i++;
                        }
                        // If move to the down
                    } else {
                        if (selectedCellsY > rowsHeight.value[bottomRightCell[0] + i]) {
                            for (let col = startCol; col <= endCol; col++) {
                                arrCells.value[bottomRightCell[0] + i][col].selected = true;
                            }
                            selectedCellsY -= rowsHeight.value[bottomRightCell[0] + i];
                            i++;
                            checkSelectedCells();
                        } else {
                            for (let col = startCol; col <= endCol; col++) {
                                arrCells.value[bottomRightCell[0] + i][col].selected = false;
                            }
                            selectedCellsY += rowsHeight.value[bottomRightCell[0] + i];
                            i--;
                        }
                    }

                }
            }
        }
        checkSelectedCells();
    }
    document.addEventListener("mousemove", mouseMove);
    document.onmouseup = () => {
        document.removeEventListener("mousemove", mouseMove);
        document.onmouseup = null;
        let r = 0;
        let col = 0;
        let isStartDuplicate = false;
        arrCells.value.forEach((row) => {
            r = isStartDuplicate ? r + 1 : 0;
            col = 0;
            row.forEach(cell=>{
                if (cell.selected) {
                    if (endCol - startCol + 1 <= col) {
                        col = 0
                    }
                    if (endRow - startRow + 1 <= r) {
                        r = 0
                    }
                    cell.content = selectedCellsValue[r][col];
                    col++;
                    cell.selected = false;
                    isStartDuplicate = true
                }
            });

        });
    };
}
const onDuplicateSelectMove = (e:Event)=>{



}

</script>

<style lang="scss" scoped>
.header {
  display: flex;
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
      &.selectionDark {
        background-color: rgba(0, 0, 0, 0.1);
      }

      &.active {
        outline: 2px solid #008200;
        background-color: transparent;
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

        .duplicate-selection {
            position: absolute;
            z-index: 2;
            right: -5px;
            bottom: -5px;
            width: 7px;
            height: 7px;
            border: 1px solid #fff;
            background-color: #000;
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
