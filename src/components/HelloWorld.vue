<template>
  <div>
    <div>价格:{{specPrice}}</div>
    <div v-for="(item, index) in specList" :key="index">
      <span>{{ item.type }}</span>
      <div class="item-list">
        <span
          v-for="(childItem, childIndex) in item.specChildList"
          :key="childIndex"
          :class="{
            selected: childItem.selected,
            disabled: childItem.disabled,
          }"
          class="text"
          @click="selectSpec(childItem, item)"
        >
          {{ childItem.name }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      specPrice:'0',
      originalSpecList:[],
      specList: [],
      specSelected: [],
    };
  },
  mounted: function() {
    // this.originalSpecList = [{
    //   price:12,
    //   stock:10,
    //   specificationItems:[
    //     {
    //       type:"颜色",
    //       name:"白色"
    //     },
    //     {
    //       type:"尺寸",
    //       name:"13"
    //     }
    //   ]
    // },{
    //   price:12,
    //   stock:10,
    //   specificationItems:[
    //     {
    //       type:"颜色",
    //       name:"红色"
    //     },
    //     {
    //       type:"尺寸",
    //       name:"13"
    //     }
    //   ]
    // },{
    //   price:12.5,
    //   stock:10,
    //   specificationItems:[
    //     {
    //       type:"颜色",
    //       name:"白色"
    //     },
    //     {
    //       type:"尺寸",
    //       name:"10"
    //     }
    //   ]
    // },{
    //   price:10,
    //   stock:0,
    //   specificationItems:[
    //     {
    //       type:"颜色",
    //       name:"红色"
    //     },
    //     {
    //       type:"尺寸",
    //       name:"10"
    //     }
    //   ]
    // }]
    this.originalSpecList = this.specBuild()
    this.specList = this.groupSkuProp(this.originalSpecList)
  },
  methods: {
    specBuild(){
      let originalSpecList = [];
      for (let i = 0; i < 4; i++) {
        const temp = [];
        const ch = String.fromCharCode(65+i)
        for (let j = 0; j < 10; j++) {
          const r =  this.addSpecification({
            type:ch,
            name:ch+j
          },originalSpecList)
          temp.push(...r)
        }
        originalSpecList = temp;
      }

      return originalSpecList;
    },
    addSpecification (itemValue,originalSpecList) {
      // 新加组合
      const newCombinations = []
      if(originalSpecList.length > 1){
        for (const specification of originalSpecList) {
          newCombinations.push([...specification.specificationItems,itemValue])
        }
      }else{
        newCombinations.push([itemValue])
      }  
      return newCombinations.map(x=>{
        return {
          price: Math.floor(Math.random() * 100),
          stock: Math.floor(Math.random() * 100),
          specificationItems: [...x]
        }
      })
    },
    groupSkuProp(specifications) {
      const specList = []
      for (const specification of specifications) {
        for (const item of specification.specificationItems) {
          let specificationItem = specList.find((x) => x.type === item.type);
          if (specificationItem) {
            let specificationchildItem = specificationItem.specChildList.find((x) => x.name === item.name);
            if (specificationchildItem) {
              specificationchildItem.specifications.push(specification);
            } else {
              specificationItem.specChildList.push({
                name: item.name,
                specifications: [specification],
              });
            }
          } else {
            specList.push({
              type: item.type,
              specChildList: [
                {
                  name: item.name,
                  specifications: [specification],
                },
              ],
            });
          }
        }
      }
      return specList;
    },
    selectSpec(childItem, specItem) {
        if (childItem.disabled) return;
        let isSelect = true;
        const index = this.specSelected.findIndex(x => x.type === specItem.type);
        if (index === -1) {
            this.specSelected.push({ type: specItem.type, name: childItem.name });
        } else {
            if (this.specSelected[index].name === childItem.name) {
                this.specSelected.splice(index, 1);
                isSelect = false;
            } else {
                this.specSelected[index] = { type: specItem.type, name: childItem.name }
            }
        }

        const list = specItem.specChildList;

        list.forEach(item => {
            this.$set(item, 'selected', false);
        })

        this.$set(childItem, 'selected', isSelect);

        this.setDisabled();
        this.showSelectInfo();   
    },
    setDisabled() {
        for (const spec of this.specList) {
            const selected = this.specSelected.filter(x => spec.type !== x.type);
            for (const item of spec.specChildList) {
                const stockZero = item.specifications
                    .filter(x => !selected.some(
                        s => !x.specificationItems.some(
                            ss => ss.type === s.type && ss.name === s.name)))
                    .every(s => s.stock <= 0);
                this.$set(item, 'disabled', stockZero);
            }
        }
    },
    showSelectInfo(){
        const specifications = this.findSelectSpec();

        if (specifications.length > 1) {
            specifications.sort((a, b) => {
                return a.price - b.price;
            })
            let min = specifications[0];
            let max = specifications[specifications.length - 1];
            this.specPrice = `${min.price}-${max.price }`;
        }
        else if (specifications.length === 1) {
            const sf = specifications[0];
            this.specPrice = sf.price;
        }
    },
    findSelectSpec() {
        let specifications = this.originalSpecList;

        for (const selected of this.specSelected) {
            specifications = specifications.filter(
                x => x.specificationItems.some(
                    ss => ss.type === selected.type &&
                        ss.name === selected.name));
        }

        return specifications;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.item-list {
    padding: 10px 0 0;
    display: flex;
    flex-wrap: wrap;
}

.text {
    display: flex;
    align-items: center;
    justify-content: center;
    background: #eee;
    margin-right: 20px;
    margin-bottom: 10px;
    border-radius: 10px;
    min-width: 20px;
    height: 20px;
    padding: 0 20px;
    font-size: 14px;
    color: #606266;
}
.selected {
    background: #fbebeb;
    color: #6395fa;
}
.disabled {
    border-style:dotted;
    background: #fff;
    color: #ddd;
} 
</style>
