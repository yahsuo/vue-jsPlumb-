<template>
    <div href="#" data-template="tpl-data" role="button" class="btn-controler">
        <Tooltip :content="id" :transfer="isTransfer">
            <Alert show-icon closable class="alert-bg">
                <Icon type="logo-buffer" slot="icon"></Icon>
                数据组件{{id}}
            </Alert>
        </Tooltip>
    </div>
</template>
<script>
    export default {
        name: "Data",
        data(){
            return{
                //这个属性可以将Tooltip上的textContent变为div层 解决了ui.draggable[0].textContent重复出现的问题
                isTransfer:true
            }
        },
        props: {
            id: {
                type: String,
                required: true
            }
        },
        //此处应使用mounted进行渲染 因为该方法在模板渲染成html后调用，通常是初始化页面完成后，再对html的dom节点进行一些需要的操作
        //而created在模板渲染成html前调用，即通常初始化某些属性值，然后再渲染成视图
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
