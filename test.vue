<template>
    <div>
        <div class="banner" v-bind:style="{height:bannerHright + 'px'}" ref="rangemy">
            <div class="coverBanner" v-bind:style="{height:value + 'px'}"></div>
            <p class="circle" v-bind:style="{bottom:value + 'px'}" 
            @touchstart.stop.prevent="touchstart" 
            @touchmove.stop.prevent="touchmove" 
            @touchend.stop.prevent="touchend"
            ></p>
            {{value}}
        </div>
    </div>
</template>

<script>
document.querySelector
export default {
    data (){
        return {
            value:40,
            bannerHright:120
        }
    },
    created(){
        
    },
    methods: {
        touchstart(e){
            console.log()
        },
        touchmove(e){
            // this.value = e.touches[0].screenY - e.touches[0].clientY;
            // this.setHeight()
            // this.x=event.offsetX;
            // this.y=event.offsetY
            // e.touches[0].clientY 
            let va = this.bannerHright - e.touches[0].clientY + this.$refs.rangemy.offsetTop ;
            this.value = va > this.bannerHright?this.bannerHright:va;
            
            if(va > this.bannerHright){
                this.value = this.bannerHright
            }else if(va < 0){
                this.value = 0
            }else{
                this.value = va
            }
        },
        touchend(e){
            // this.value = e.touches[0].clientY;
            // this.setHeight()
        },
    }
}
</script>


<style scoped>
.banner{
    width: 6px;
    height: 0;
    background: #e4e7ed;
    position: fixed;
}
.circle{
    position: absolute;
    border-radius: 50%;
    left: 50%;
    transform: translate(-50%,0);
    bottom: 0px;
    width: 18px;
    height: 18px;
    border: 2px solid #409eff;
    background-color: #fff;
    /* transition: .2s; */
    user-select: none;
}
.coverBanner{
    position: absolute;
    width: 100%;
    height: 0px;
    background: #409eff;
    bottom: 0;
}

</style>

