<template>
  <div class="header">
    <div class="cellName">{{ tHead[colActive].name }}{{ rowActive + 1 }}</div>
    <div class="input">
      <input
        type="text"
        name=""
        id=""
        :value="arrCells[rowActive][colActive].content"
        @input="arrCells[rowActive][colActive].content = $event.target.value"
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
          @keypress.enter.prevent="onNextCell(col, $event)"
          @keyup.down.prevent="onNextCell(col, $event)"
          @keyup.up.prevent="onUpCell(col)"
          @keyup.left.prevent="onLeftCell(col)"
          @keyup.right.prevent="onRightCell(col)"
        >
          <div
            class="cell-content"
            :contenteditable="col.editable"
            @input="col.content = $event.target.textContent"
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

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    cols: { type: Number, default: 26 },
    rows: { type: Number, default: 10 },
  },
  data() {
    return {
      arrCells: [],
      tHead: [],
      rowsHeight: [],
      colActive: 0,
      rowActive: 0,
    };
  },
  methods: {
    updateCell(e: Event) {
      this.activeCellInput = e.target.value;
      this.$refs.rows[this.rowIndex].children[this.colIndex + 1].textContent =
        this.activeCellInput;
    },
    activeCell(cell, e) {
      let cellActive;
      for (let i = 0; i < this.arrCells.length; i++) {
        cellActive = this.arrCells[i].find((el) => el.active == true);
        if (cellActive) {
          cellActive.active = false;
          cellActive.editable = false;
          break;
        }
      }
      this.colActive = cell.col;
      this.rowActive = cell.row;

      cell.active = true;
      cell.editable = true;
      this.$nextTick(() => {
        e.target.focus();
      });
    },
    onNextCell(cell, e) {
      if (this.rowActive + 1 < this.rows) {
        cell.active = false;
        cell.editable = false;

        this.rowActive += 1;
        this.arrCells[this.rowActive][this.colActive].active = true;
        this.arrCells[this.rowActive][this.colActive].editable = true;
        this.$nextTick(() => {
          this.$refs.refRows[this.rowActive].children[
            this.colActive + 1
          ].firstChild.focus();
        });
      }
    },
    onKeypress(cell) {
      cell.active = false;
      cell.editable = false;
      this.arrCells[this.rowActive][this.colActive].active = true;
      this.arrCells[this.rowActive][this.colActive].editable = true;
      this.$nextTick(() => {
        this.$refs.refRows[this.rowActive].children[
          this.colActive + 1
        ].firstChild.focus();
      });
    },
    onUpCell(cell) {
      if (this.rowActive > 0) {
        this.rowActive -= 1;
        this.onKeypress(cell);
      }
    },
    onLeftCell(cell) {
      if (this.colActive > 0) {
        this.colActive -= 1;
        this.onKeypress(cell);
      }
    },
    onRightCell(cell) {
      if (this.colActive + 1 < this.cols) {
        this.colActive += 1;
        this.onKeypress(cell);
      }
    },
    colResize(i: number, event) {
      const col = event.target.parentElement;
      const colWidth = event.target.parentElement.offsetWidth;
      const colX = event.pageX;

      const mouseMove = (event: Event) => {
        this.tHead[i].width = colWidth - (colX - event.pageX);
        this.tHead[i].width = colWidth - (colX - event.pageX);
      };
      document.addEventListener("mousemove", mouseMove);

      document.onmouseup = function () {
        document.removeEventListener("mousemove", mouseMove);
        col.onmouseup = null;
      };
      col.ondragstart = function () {
        return false;
      };
    },
    rowResize(i: number, event) {
      const row = event.target.parentElement;
      const rowHeight = event.target.parentElement.offsetHeight;
      const rowY = event.pageY;

      const mouseMove = (event: Event) => {
        this.rowsHeight[i] = rowHeight - (rowY - event.pageY);
        // row.style.maxWidth = rowWidth - (rowX - event.pageX) + "px";
      };
      document.addEventListener("mousemove", mouseMove);
      document.onmouseup = function () {
        document.removeEventListener("mousemove", mouseMove);
        row.onmouseup = null;
      };
      row.ondragstart = function () {
        return false;
      };
    },
    selectCells(e) {
      let startSelectedX = e.pageX;
      let startSelectedY = e.pageY;

      const mouseMove = (event: Event) => {
        let selectedCellsX = event.pageX - startSelectedX;
        let selectedCellsY = event.pageY - startSelectedY;
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
            this.arrCells.forEach((row) => {
              if (row[this.colActive].selected) {
                row[this.colActive].selected = false;
              }
            });
            if (this.colActive + i < this.cols && this.colActive + i > 0) {
              if (selectedCellsX < 0) {
                if (
                  Math.abs(selectedCellsX) -
                    this.tHead[this.colActive + i - 1].width >
                  0
                ) {
                  this.arrCells[this.rowActive][
                    this.colActive + i - 1
                  ].selected = true;
                  selectedCellsX += this.tHead[this.colActive + i - 1].width;
                  i--;
                  checkSelectedCells();
                } else {
                  this.arrCells[this.rowActive][
                    this.colActive + i - 1
                  ].selected = false;
                  selectedCellsX -= this.tHead[this.colActive + i - 1].width;
                  i++;
                }
              } else if (
                selectedCellsX - this.tHead[this.colActive + i].width >
                0
              ) {
                this.arrCells[this.rowActive][this.colActive + i].selected =
                  true;
                selectedCellsX -= this.tHead[this.colActive + i].width;
                i++;
                checkSelectedCells();
              } else {
                this.arrCells[this.rowActive][this.colActive + i].selected =
                  false;
                selectedCellsX += this.tHead[this.colActive + i].width;
                i--;
              }
            }
          } else {
            this.arrCells[this.rowActive].find((cell) => {
              cell.selected = false;
            });

            if (this.rowActive + i < this.rows && this.rowActive + i > 0) {
              if (selectedCellsY < 0) {
                if (
                  Math.abs(selectedCellsY) >
                  this.rowsHeight[this.rowActive + i - 1]
                ) {
                  this.arrCells[this.rowActive + i - 1][
                    this.colActive
                  ].selected = true;
                  selectedCellsY += this.rowsHeight[this.rowActive + i - 1];
                  i--;
                  checkSelectedCells();
                } else {
                  this.arrCells[this.rowActive + i - 1][
                    this.colActive
                  ].selected = false;
                  selectedCellsY -= this.rowsHeight[this.rowActive + i - 1];
                  i++;
                }
              } else {
                if (selectedCellsY > this.rowsHeight[this.rowActive + i]) {
                  this.arrCells[this.rowActive + i][this.colActive].selected =
                    true;
                  selectedCellsY -= this.rowsHeight[this.rowActive + i];
                  i++;
                  checkSelectedCells();
                } else {
                  this.arrCells[this.rowActive + i][this.colActive].selected =
                    false;
                  selectedCellsY += this.rowsHeight[this.rowActive + i];
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
        this.arrCells[this.rowActive].find((cell) => {
          if (cell.selected) {
            cell.content =
              this.arrCells[this.rowActive][this.colActive].content;
            cell.selected = false;
          }
        });
        this.arrCells.forEach((row) => {
          if (row[this.colActive].selected) {
            row[this.colActive].content =
              this.arrCells[this.rowActive][this.colActive].content;
            row[this.colActive].selected = false;
          }
        });
        // const duplicateContent = () => {
        //   if (
        //     this.colActive + i + 1 <= this.cols &&
        //     selectedCells - this.tHead[this.colActive + i].width > 0
        //   ) {
        //     // Копіювання тексту до останньо комірки
        //     if (this.colActive + i + 1 == this.cols) {
        //       this.arrCells[this.rowActive][this.colActive + i].content =
        //         this.arrCells[this.rowActive][this.colActive].content;
        //       this.arrCells[this.rowActive][this.colActive + i].selected =
        //         false;
        //       return;
        //     }
        //     this.arrCells[this.rowActive][this.colActive + i].content =
        //       this.arrCells[this.rowActive][this.colActive].content;
        //     this.arrCells[this.rowActive][this.colActive + i].selected = false;

        //     selectedCells -= this.tHead[this.colActive + i].width;
        //     i++;
        //     duplicateContent();
        //   } else {
        //     return;
        //   }
        // };
        // duplicateContent();
      };
      e.target.ondragstart = function () {
        return false;
      };
    },
  },
  computed: {
    columnLetter() {
      const letterArr: string[] = [
        "A",
        "B",
        "C",
        "D",
        "E",
        "F",
        "G",
        "H",
        "I",
        "J",
        "K",
        "L",
        "M",
        "N",
        "O",
        "P",
        "Q",
        "R",
        "S",
        "T",
        "U",
        "V",
        "W",
        "X",
        "Y",
        "Z",
      ];
      for (let i = 0; i < this.cols; i++) {
        if (i < 26) {
          this.tHead.push({ name: letterArr[i], width: 100 });
        } else {
          const colLetter =
            letterArr[Math.floor(i / 26) - 1] + letterArr[i % 26];
          this.tHead.push({ name: colLetter, width: 100 });
        }
      }
      return this.tHead;
    },
  },
  created() {
    for (let i = 0; i < this.rows; i++) {
      this.arrCells.push([]);
      this.rowsHeight.push(30);
      for (let j = 0; j < this.cols; j++) {
        this.arrCells[i].push({
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
    this.arrCells[0][0].active = true;
    this.arrCells[0][0].editable = true;
    this.$nextTick(() => {
      this.$refs.refRows[0].children[1].firstChild.focus();
    });
    this.columnLetter;
  },
});
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
}
table {
  width: max-content;
  border-collapse: collapse;
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
