<template>
    <div>
        <navbar @searchEvent="search"></navbar>
        <b-container fluid>
        <b-row class="">
                <b-col  class="col-2 pt-5">
                    <b-container>

                        <b-row class="pb-2">
                            <H3>Filter</H3>
                        </b-row>
                        <b-row class="pb-2 border-top">
                            <H6 class="pt-1">Price Range</H6>
                            <b-input-group size="sm" prepend="$">
                                <b-form-input v-model="priceLow" class="mr-2"></b-form-input>-
                                <b-form-input v-model="priceHigh" class="ml-2"></b-form-input>
                            </b-input-group>
                        </b-row>
                        <b-row class="pt-1 pb-2 border-top">
                            <H6 class="pt-1">Tags</H6>
                            <b-form-select v-model="tagSelected" :options="tags" size="sm" ></b-form-select>
                            <!--                            <b-form-tags
                                                                input-id="tags-pills"
                                                                v-model="filterTag"
                                                                tag-variant="secondary"
                                                                tag-pills
                                                                size="sm"
                                                                separator=" "
                                                                placeholder="Remove Unwanted Tag"
                                                        ></b-form-tags>-->
                        </b-row>
                        <b-row class="pt-1 pb-2">
                            <b-button block variant="outline-success" @click="onFilterApplyClicked()">Apply</b-button>
                            <b-button block variant="outline-danger" @click="onFilterResetClicked()">Reset</b-button>

                        </b-row>

                    </b-container>

                </b-col>
                    <b-col class="col-10">
                        <b-card-group  columns>
                            <div v-for="i in items" :key="i.id" class="card h-100">
                                <b-img @click="onItemDetailClick(i.id)" class="card-img-top"  :src="i.thumb" fluid alt="Fluid image"></b-img>
                                <div class="card-body">
                                    <h5 class="card-title font-weight-bold"><a href="#" @click="onItemDetailClick(i.id)">{{i.title}}</a></h5>
                                    <h6 class="card-subtitle mb-2 text-info">{{'$'+i.price}}</h6>
                                    <p class="card-text">{{i.desc}}</p>
                                </div>
                            </div>
                        </b-card-group>
                    </b-col>

            </b-row>

        </b-container>
    </div>

</template>

<script>
    import navbar from "@/components/navbar";
    export default {
        name: "itemBrowse",
        components: {navbar},
        data(){
            return{
                filterTag: [],
                items: [],
                keyword:'',
                tagSelected:null,
                priceLow:null,
                priceHigh:null,
                tags:[]
            }
        },
        mounted() {
            let keyword = this.$route.query.keyword;
            console.log('keyword:' + keyword);
            if(keyword !== undefined){
                if(keyword.length > 0 ){
                    this.keyword = keyword
                }
            }
            //console.log('this keyword:' + keyword);
            this.fetchData({FuzzyTitle : this.keyword });
        },
        methods: {
            search(keyword){
                if(keyword === null){
                    this.keyword = '';
                } else{
                    this.keyword = keyword;
                }
                this.fetchData({FuzzyTitle : this.keyword });
                //console.log('navbar keyword:'+ keyword);
                //console.log('local keyword:'+ this.keyword);

            }
            ,
            onItemDetailClick(id) {
                console.log(id);
                this.$router.push({ path: "/detail/"+id }).then(this.$forceUpdate());
            },
            onFilterApplyClicked(){
                let minPrice = 0;
                let maxPrice = 999999999999;
                if(this.priceLow !== null){
                    minPrice = parseFloat(this.priceLow);
                }
                if(this.priceHigh !== null){
                    maxPrice = parseFloat(this.priceHigh);
                }
                let json = {FuzzyTitle : this.keyword, PriceLow:minPrice, PriceHigh:maxPrice };
                if(this.tagSelected !== null){
                    json = {FuzzyTitle : this.keyword, Tags: [this.tagSelected], PriceLow:minPrice, PriceHigh:maxPrice };
                }
                this.fetchData(json);
            },
            onFilterResetClicked(){
                let json = {FuzzyTitle : '' };
                this.tagSelected = null;
                this.priceLow = null;
                this.priceHigh = null;
                this.fetchData(json);
            },
            fetchData(json){
                //let json = {FuzzyTitle : this.keyword };
                console.log('JSON:'+JSON.stringify(json));
                var link = 'http://165.232.138.223:8080/item/get';
                //console.log('link:' + link);

                fetch(link, {
                    method: 'POST',
                    body: JSON.stringify(json),
                    headers: {
                        'Content-Type': 'application/json'
                    },
                }).then(response => response.json())
                    .then(data => {
                        console.log(data);
                        this.items = this.preProcess(data.Items);
                    }).catch(error=>{
                        console.log(error);
                    }
                )
            },
            preProcess(data){
                console.log(data);
                if(data === null){return }
                let output = [];
                var tag = [];
                for(let i=0; i< data.length; i++){
                    let record = data[i];
                    let title = "No title";
                    let desc = "No description provided";
                    let thumb = this.getEmptyImage();
                    let price = 0;
                    let id = null;
                    if(record.Title.length !== 0 ){
                        title = record.Title;
                    }
                    if(record.Description.length !== 0 ){
                        desc = record.Description;
                    }
                    if(record.NewPrice !== 0 ){
                        price = record.NewPrice;
                    }
                    if(record.Thumbnails !== null){
                        thumb = "data:image/png;base64," + record.Thumbnails;
                    }else {
                        thumb = "data:image/png;base64," + thumb;
                    }
                    if(record.ObjId !== 0 ){
                        id = record.ObjId;
                    }
                    if(record.Tags.length > 0){
                        for(let j =0; j < record.Tags.length ; j++){
                            tag.push(record.Tags[j]);
                            //console.log("pushed"+tag);
                        }
                    }

                    output.push({id:id, title:title, desc:desc, thumb:thumb, price:price})
                }
                tag = [...new Set(tag)];
                let tempTags = [];
                for(let i = 0; i < tag.length ; i++){
                    let tagInArray = {value:tag[i], text:tag[i]}
                    tempTags.push(tagInArray);
                }
                this.tags = tempTags;
                this.filterTag = [...new Set(tag)];
                return output;

            },
            getEmptyImage(){
                return "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"
            }
        }
    }
</script>

<style>
    .card-img-top {
        width: 100%;
        height: 12vw;
        object-fit: cover;
    }
    a { text-decoration: none; }

</style>