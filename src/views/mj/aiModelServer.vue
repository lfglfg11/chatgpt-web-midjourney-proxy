<script setup lang="ts">
//NInfiniteScroll
import {NSelect, NInput,NSlider, NButton, useMessage,NTag,NEmpty,NModal,NDivider
} from "naive-ui"
//import type { SelectRenderLabel, SelectRenderTag } from 'naive-ui'
import { ref ,computed,watch, onMounted,h} from "vue";
import { SvgIcon } from '@/components/common'
import { gptFetch, mlog } from "@/api";

const st= ref({ server:'',isShow:false,isLoadData:0 ,"search":''});
const ms= useMessage();
  
const emit= defineEmits(['success']);
interface modelType{
    model:string
    //maxToken:Number
}
interface modelGroup{
    name:string
    key:string[]
    data:modelType[]
    class?:string[]
    icon?:string
    isClosed?:boolean
    matcher?:(modelId: string) => boolean
}

const mGroup= ref<modelGroup[]>([])

const findGroupForModelId = (modelId: string) => {
    for (const group of mGroup.value) {
        if (group.matcher && group.matcher(modelId)) return group
        if (group.key?.length && group.key.some(k => modelId.includes(k))) return group
    }
    return mGroup.value.find(g => g.name === 'Other')
}

const loadModel=async ()=>{
    try {
         const modelsData = await gptFetch('/v1/models');
          mlog('asdsd>> ', modelsData )
        modelsData.data.forEach((v:any) => {
              mlog('vv>> ',v )
              const modelId = v?.id as string | undefined
              if (!modelId) return
              const group = findGroupForModelId(modelId)
              group?.data.push({ model: modelId })
        });
        st.value.isLoadData=1
    } catch (error) {
        st.value.isLoadData=-1
        ms.error('Loading Models Error!')
    } 
}

const initGroup=()=>{
    // 以下三类用于将 jimeng / banana / jimeng-video 从 Other 分离出来
    mGroup.value.push({
        name: '即梦绘图',
        key: [],
        data: [],
        matcher: (id: string) => id.startsWith('jimeng-') && !id.startsWith('jimeng-video')
    })
    mGroup.value.push({
        name: '谷歌绘图',
        key: [],
        data: [],
        matcher: (id: string) => id.startsWith('nano-banana')
    })
    mGroup.value.push({
        name: '视频生成',
        key: [],
        data: [],
        matcher: (id: string) => id.startsWith('jimeng-video') || id.startsWith('sora') || id.startsWith('veo')
    })
        mGroup.value.push({
        name: 'GLM',
        key: [],
        data: [],
        matcher: (id: string) => id.startsWith('GLM')
    })
    // {name:'OpenAi',key:['gpt-'],data:[]}
    // ,{name:'OpenAi O',key:['o1-','o3-'],data:[]}
    // ,{name:'Deepseek',key:['deepseek'],data:[]}
    mGroup.value.push( {name:'Deepseek',key:['deepseek'],data:[],icon:"arcticons:deepseek"} )
    mGroup.value.push({name:'OpenAi',key:['gpt-'],data:[],'icon':'ri:openai-fill'})
    mGroup.value.push( {name:'OpenAi O',key:['o1','o3'],data:[],'icon':'ri:openai-fill'} )
    mGroup.value.push( {name:'Claude',key:['claude','c-3'],data:[],icon:"ri:claude-fill"} )
    mGroup.value.push( {name:'Gemini',key:['gemini'],data:[],icon:"cbi:gemini"} )
    mGroup.value.push( {name:'Grok',key:['grok'],data:[],icon:"token:xai"} )

    mGroup.value.push( {name:'Other',key:[],data:[],isClosed:true } )
}

onMounted(()=>{
 initGroup();
 loadModel()
})

const successClick=(md:any)=> {
    emit('success', md)
    st.value.isShow=false
}

//const usageData = await gptFetch(urlUsage);
const modellist = computed(() => {
    let rz: any[]=[];
    for(let o of mGroup.value){
        if(o.data.length<=0) continue
        for(let v of o.data){
         rz.push({label:v.model,value:v.model})
        }
    }
    return rz;
});
const abc=()=>{
    //console.log('abc>> ',st.value.search)
    if(st.value.search=='') return;
    for(let o of mGroup.value){
        if(o.data.length<=0) continue
        for(let v of o.data){
         //rz.push({label:v.model,value:v.model})
         if(v.model==st.value.search) successClick( v)
        }
    }
}
</script>
<template>
<div @click="st.isShow=true">
<NTag  type="primary" round size="small" :bordered="false" class="!cursor-pointer">
            {{ $t('mj.server_load') }}  </NTag>
 
</div>
<NModal  v-model:show="st.isShow"  preset="card"  :title="$t('mj.model_select')" class="!max-w-[620px]" @close="st.isShow=false" >
     <NEmpty v-if="st.isLoadData==0">Loading....</NEmpty>
     <NEmpty v-else-if="st.isLoadData==-1">Loaded Fail ....</NEmpty>
     <div   class=" overflow-y-auto max-h-[400px]" v-else >
        <div>
            
            <n-select v-model:value="st.search" @update:value="abc" clearable :options="modellist" size="small" placeholder="search and select your model"  filterable   />
       
        </div>
        <div v-for="mg in mGroup">
            <template v-if="mg.data.length>0">
            <div class="  relative"  >
                <n-divider title-placement="left">
                    <div class="flex justify-start items-center space-x-2" @click="mg.isClosed=!mg.isClosed">
                        <SvgIcon :icon="mg.icon" v-if="mg.icon"/>
                        <span>{{ mg.name }} </span>
                    </div>
                </n-divider>
                <div class="absolute right-3 top-[5px] rounded-full bg-white/10 cursor-pointer " @click="mg.isClosed=!mg.isClosed" :class="{'rotate-180':mg.isClosed}">
                   <SvgIcon icon="ri:arrow-up-s-line" />
                </div>
                <div class="absolute right-10 top-[2px]"  @click="mg.isClosed=!mg.isClosed">
                    <NTag  size="small" round   :bordered="false">
                        <span class=" cursor-pointer">{{ mg.data.length }}</span>
                    </NTag>
                </div>
            </div>
            <div v-if="mg.data.length>0 && !mg.isClosed" class="grid   gap-2 grid-cols-2">
                <div v-for="md in mg.data" >
                    <!-- <template v-if="st.search=='' || md.model.includes(st.search)"></template> -->
                   <NTag type="info" size="small" round>
                     <span class="cursor-pointer" @click="successClick(md)" >{{ md.model }}</span>
                   </NTag>
                   
                </div>
            </div>
            </template>
        </div>
     </div>
 </NModal>
</template>