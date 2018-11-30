<template>
    <div href="#" data-template="tpl-model" role="button" class="btn-controler">
        <Tooltip :content="id" :transfer="isTransfer">
            <Alert class="alert-bg" show-icon closable>
                <Icon type="md-bulb" slot="icon"></Icon>
                模型组件{{id}}
            </Alert>
        </Tooltip>
    </div>
</template>

<script>
    export default {
        name: "Model",
        props: {
            id: {
                type: String,
                required: true
            }
        },
        data(){
            return{
                //这个属性可以将Tooltip上的textContent变为div层 解决了ui.draggable[0].textContent重复出现的问题
                isTransfer:true
            }
        },
        mounted() {
            //由于该组件是动态生成的 所以需要一个个添加可拖动属性
            $('.btn-controler').draggable({
                helper: 'clone',
                scope: 'ss',
            })
        }
    }
</script>

<style scoped>
    .btn-controler {
        z-index: 60;
    }

    .alert-bg {
        width: 150px;
        background: #f7f3ff;
        /*如果是中文 设置文字超出范围不断行*/
        white-space: nowrap;
        overflow: hidden;
        /*设置多余文本隐藏显示；*/
        text-overflow: ellipsis;
    }
</style>