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
                    @keyup="()=>onNextCell"
            />
        </div>
    </div>
</template>

<script setup lang="ts">

interface Cell {
    content: string,
    active: boolean,
    selected: boolean,
    editable: boolean,
    selectCell: boolean,
    width: number,
    row: number,
    col: number,
    mathExp:string,
}
interface Props{
    rowActive:Number,
    colActive:Number,
    arrCells: Cell[][],
    headerLetter: [{name:string,width:number}],
    onNextCell:Function

}
// const props = defineProps({
//     rowActive: {type: Number, required: true},
//     colActive: {type: Number, required: true},
//     headerLetter: {type: Array, required: true},
//     arrCells: {type: Array, required: true},
//     onNextCell: {type: Function},
// })
const props = defineProps<{
    rowActive:number,
    colActive:number,
    arrCells: Cell[][],
    headerLetter: [{name:string,width:number}],
    onNextCell:Function
}>()
const heandlerClick = ()=>{
    console.log('1')
}
</script>

<style scoped lang="scss">
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

</style>