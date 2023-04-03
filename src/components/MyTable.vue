<template>
    <div class="header">
        <div class="cellName"></div>
        <div class="input">
            <input
              type="text"
              name=""
              id=""
              :value="activeCellInput"
              @input="updateCell"
            />
        </div>
    </div>
    <div class="table-wrapper">
        <table>
            <thead>
            <tr>
                <th></th>
                <th
                  v-for="(col, i) in cols"
                  :key="i"
                  ref="cols"
                  @mousedown="onMousedown"
                >
                    {{ columnLetter(i) }}
                    <span class="col-resize" @mousedown="colResize"></span>
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(row, rowInd) in rows" :key="rowInd" ref="rows">
                <td>
                    {{ rowInd + 1 }}
                    <span class="row-resize" @mousedown="rowResize"></span>
                </td>
                <td
                  @input="activeCellInput = $event.target.textContent"
                  v-for="(col, colInd) in cols"
                  :key="colInd"
                  @click="activeCell(rowInd, colInd, $event)"
                  @keypress.enter.prevent="onNextCell(rowInd, colInd, $event)"
                  @keyup.right="onRightCell(rowInd, colInd, $event)"
                  @keyup.left.prevent="onLeftCell(rowInd, colInd, $event)"
                  @keyup.down.prevent="onNextCell(rowInd, colInd, $event)"
                  @keyup.up.prevent="onUpCell(rowInd, colInd, $event)"
                  @mousedown="selectCells"
                >
                    <!-- <span
                      class="select-cells"
                      @mousedown="selectCells"
                      v-if="rowIndex == rowInd && colIndex == colInd"
                    ></span> -->
                </td>
            </tr>
            </tbody>
            <!-- <tfoot>
                <template v-if="true">
                  <tr>
                    <th></th>
                    <th v-for="(col, i) in cols" :key="i"></th>
                  </tr>
                </template>
                <template v-if="true">
                  <tr v-for="(row, rowInd) in rows" :key="rowInd">
                    <td>{{ rowInd + 1 }}</td>
                    <td
                      v-for="(col, colInd) in cols"
                      :key="colInd"
                      @click="activeCell(rowInd, colInd, $event)"
                    >
                      {{ rowInd + ", " + colInd }}
                    </td>
                  </tr>
                </template>
              </tfoot> -->
        </table>
    </div>
</template>

<script lang="ts">
import {defineComponent} from "vue";

export default defineComponent({
  props: {
    cols: {type: Number, default: 26},
    rows: {type: Number, default: 10},
  },
  data() {
    return {
      activeCellInput: "",
      rowIndex: 0,
      colIndex: 0,
      lastActiveCell: "",
    };
  },
  methods: {
    updateCell(e: Event) {
      this.activeCellInput = e.target.value;
      this.$refs.rows[this.rowIndex].children[this.colIndex + 1].textContent =
        this.activeCellInput;
    },
    activeCell(rowI: number, colI: number, e: Event) {
      if (this.lastActiveCell.classList?.contains("active")) {
        this.lastActiveCell.classList.remove("active");
        this.lastActiveCell.setAttribute("contenteditable", "false");
        (this.$refs.cols as HTMLCollection)[this.colIndex].classList.remove(
          "active-col"
        );
        (this.$refs.rows as any)[this.rowIndex].firstChild.classList.remove(
          "active-row"
        );
      }

      (e.target as Element).classList.add("active");
      (e.target as Element).setAttribute("contenteditable", "true");
      (e.target as HTMLElement).focus();
      (this.$refs.cols as HTMLCollection)[colI].classList.add("active-col");
      (this.$refs.rows as any)[rowI].firstChild.classList.add("active-row");

      this.activeCellInput = (e.target as Element).textContent;
      this.rowIndex = rowI;
      this.colIndex = colI;
      this.lastActiveCell = e.target;
    },

    columnLetter(i: number) {
      const letterArr1: string[] = [
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

      if (i < 26) {
        return letterArr1[i];
      } else {
        return letterArr1[Math.floor(i / 26) - 1] + letterArr1[i % 26];
      }

      //   let letterArr: string[] = [...Array(26)].map((_, i) =>
      //     String.fromCharCode(i + 65)
      //   );
    },
    onNextCell(rowI: number, colI: number, e: Event) {
      if (rowI + 1 != this.rows) {
        this.rowIndex = rowI + 1;
        (e.target as Element).classList.remove("active");
        this.lastActiveCell.setAttribute("contenteditable", "false");

        const refRows: any = this.$refs.rows;
        refRows[rowI].firstChild.classList.remove("active-row");
        refRows[this.rowIndex].firstChild.classList.add("active-row");
        refRows[this.rowIndex].children[colI + 1].classList.add("active");
        refRows[this.rowIndex].childNodes[colI + 2].setAttribute(
          "contenteditable",
          "true"
        );
        refRows[this.rowIndex].children[colI + 1].focus();
        this.activeCellInput =
          refRows[this.rowIndex].children[colI + 1].textContent;
        this.lastActiveCell = refRows[this.rowIndex].children[colI + 1];

        // this.activeCell(rowI, colI, e);
      }
      //   console.log(rowI + 1, this.rows);
    },
    onUpCell(rowI: number, colI: number, e: Event) {
      if (rowI != 0) {
        this.rowIndex = rowI - 1;
        (e.target as Element)?.classList.remove("active");
        this.lastActiveCell.setAttribute("contenteditable", "false");

        const refRows: any = this.$refs.rows;
        refRows[rowI].firstChild.classList.remove("active-row");
        refRows[this.rowIndex].firstChild.classList.add("active-row");
        refRows[this.rowIndex].children[colI + 1].classList.add("active");
        refRows[this.rowIndex].children[colI + 1].setAttribute(
          "contenteditable",
          "true"
        );
        refRows[this.rowIndex].children[colI + 1].focus();
        this.activeCellInput =
          refRows[this.rowIndex].children[colI + 1].textContent;
        this.lastActiveCell = refRows[this.rowIndex].children[colI + 1];
      }
      //   console.log(rowI, 0);
    },

    onRightCell(rowI: number, colI: number, e: Event) {
      if (colI + 1 != this.cols) {
        this.colIndex = colI + 1;
        (e.target as Element)?.classList.remove("active");
        (e.target as Element)?.setAttribute("contenteditable", "false");

        const refs: any = this.$refs;
        refs.cols[colI].classList.remove("active-col");
        refs.cols[this.colIndex].classList.add("active-col");
        refs.rows[rowI].children[this.colIndex + 1].classList.add("active");
        refs.rows[rowI].children[this.colIndex + 1].setAttribute(
          "contenteditable",
          "true"
        );
        refs.rows[rowI].children[this.colIndex + 1].focus();
        this.activeCellInput =
          refs.rows[rowI].children[this.colIndex + 1].textContent;
        this.lastActiveCell = refs.rows[rowI].children[this.colIndex + 1];
      }
    },
    onLeftCell(rowI: number, colI: number, e: Event) {
      if (colI != 0) {
        this.colIndex = colI - 1;
        (e.target as Element)?.classList.remove("active");
        (e.target as Element)?.setAttribute("contenteditable", "false");

        const refs: any = this.$refs;
        refs.cols[colI].classList.remove("active-col");
        refs.cols[this.colIndex].classList.add("active-col");
        refs.rows[rowI].children[this.colIndex + 1].classList.add("active");
        refs.rows[rowI].children[this.colIndex + 1].setAttribute(
          "contenteditable",
          "true"
        );
        refs.rows[rowI].children[this.colIndex + 1].focus();
        this.activeCellInput =
          refs.rows[rowI].children[this.colIndex + 1].textContent;
        this.lastActiveCell = refs.rows[rowI].children[this.colIndex + 1];
      }
    },
    onMousedown(e: any) {
      function moveAt(pageX: number) {
        e.target.style.maxWidth = pageX + "px";
        e.target.style.width = pageX + "px";
      }
    },
    colResize(event: Event) {
      // console.log(event.pageX, event.pageY);
      const col = event.target.parentElement;
      const colWidth = event.target.parentElement.offsetWidth;
      const colX = event.pageX;

      document.addEventListener("mousemove", mouseMove);

      function mouseMove(event: Event) {
        col.style.width = colWidth - (colX - event.pageX) + "px";
        col.style.maxWidth = colWidth - (colX - event.pageX) + "px";
      }

      document.onmouseup = function () {
        document.removeEventListener("mousemove", mouseMove);
        col.onmouseup = null;
      };
      col.ondragstart = function () {
        return false;
      };
    },
    rowResize(event: Event) {
      // console.log(event.pageX, event.pageY);
      const row = event.target.parentElement;
      const rowHeight = event.target.parentElement.offsetHeight;
      const rowY = event.pageY;

      document.addEventListener("mousemove", mouseMove);

      function mouseMove(event: Event) {
        row.style.height = rowHeight - (rowY - event.pageY) + "px";
        // row.style.maxWidth = rowWidth - (rowX - event.pageX) + "px";
      }

      document.onmouseup = function () {
        document.removeEventListener("mousemove", mouseMove);
        row.onmouseup = null;
      };
      row.ondragstart = function () {
        return false;
      };
    },
    selectCells(event: Event) {
      const row = event.target.parentElement;
      if (event.target.classList.contains("active")) {
        console.log(row);
      }
    },
  },
  computed: {},
  mounted() {
    // this.activeCell(this.rowIndex, this.colIndex, this.lastActiveCell);
    const el: Element = (this.$refs.rows as any)[this.rowIndex].children[
    this.colIndex + 1
      ];
    el.classList.add("active");
    el.setAttribute("contenteditable", "true");
    (el as HTMLElement).focus();
    (this.$refs.cols as HTMLCollection)[this.colIndex].classList.add(
      "active-col"
    );
    (this.$refs.rows as any)[this.rowIndex].firstChild.classList.add(
      "active-row"
    );
    this.lastActiveCell = el;
  },
});
</script>

<style lang="scss" scoped>
.input {
  margin: 10px;

  input {
    width: 100%;
    font-size: 1em;
    border-radius: 10px;
    border: 1px solid #ccc;
    padding: 5px 10px;
    outline: none;
  }
}

.table-wrapper {
  width: 99vw;
  height: max-content;
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

  .active {
    outline: 2px solid #008200;
  }

  .active-col {
    background-color: rgb(0, 0, 0, 0.1);
    border-bottom: 2px solid #008200;
  }

  .active-row {
    background-color: rgb(0, 0, 0, 0.1);
    border-right: 2px solid #008200;
  }
}
</style>
