<template>
    <div class="row">
        <!-- Main Category -->
        <div class="form-group">
            <label for="mainCategory">Main Category</label>
            <vSelect
                class="form-control form-control-lg"
                id="mainCategory"
                v-model="mainCategory"
                placeholder="Please Select Main Category"
                :options="mainCategoriesWithSubCategories"
            />
        </div>
        <!-- Sub Category -->

        <div v-if="mainCategory" class="form-group mt-3">
            <label for="subCategory">Sub Category</label>

            <vSelect
                class="form-control form-control-lg"
                id="subCategory"
                v-model="subCategory"
                :options="mainCategory.children"
                :placeholder="`Please Select on sub category of ${mainCategory.label}`"
                v-on:option:selected="loadPropertiesForSelectedSubCategory"
            />
        </div>

        <!-- Properties -->
        <div v-if="subCategory && properties.length">
            <div
                v-for="prop in properties"
                :key="prop.id"
                class="form-group mt-2"
            >
                <label :for="`prop-${prop.id}`" v-text="prop.label"></label>
                <vSelect
                    class="form-control form-control-lg"
                    :options="mapOptions(prop.options)"
                    :value="prop"
                    :id="`prop-${prop.id}`"
                    :placeholder="`Please Select ${prop.label}`"
                    @option:selected="
                        (propData) => {
                            updateSelectedProperties(propData, prop);
                            updateProperties(propData, prop);
                        }
                    "
                />

                <input
                    :placeholder="`specify other ${prop.name}`"
                    class="form-control form-control-lg mt-3"
                    v-if="selectedProperties[prop.id]?.name === 'other'"
                    type="text"
                    v-model="selectedProperties[prop.id].value"
                />
            </div>
        </div>
    </div>
</template>

<script>
import vSelect from "vue-select";
import "vue-select/dist/vue-select.css";

export default {
    components: { vSelect },
    mounted() {
        this.getMainCategoriesWithSubCategories();
    },
    data() {
        return {
            mainCategoriesWithSubCategories: [],
            properties: [],

            // Selected Values
            subCategory: null,
            mainCategory: null,
            selectedProperties: [],
        };
    },

    watch: {
        mainCategory: {
            handler: function (newVal, oldVal) {
                this.subCategory = null;
                this.properties = [];
                this.selectedProperties = [];
                this.$emit("properties", this.getAllSelectedProperties);
            },
            deep: true,
        },
        subCategory: {
            handler: function (newVal, oldVal) {
                this.properties = [];
                this.selectedProperties = [];
                this.$emit("properties", this.getAllSelectedProperties);
            },
            deep: true,
        },
        selectedProperties: {
            handler: function (newVal, oldVal) {
                this.$emit("properties", this.getAllSelectedProperties);
            },
            deep: true,
        },
    },
    computed: {
        getAllSelectedProperties() {
            return {
                mainCategory: this.mainCategory,
                subCategory: this.subCategory,
                properties: this.selectedProperties,
            };
        },
    },
    methods: {
        getChildProperties(id) {
            return axios
                .get(`https://staging.mazaady.com/get-options-child/${id}`)
                .then(({ data }) => {
                    if (data && data.code == 200) {
                        const mappedChild = data.data.map((prop) => {
                            const options = [
                                ...prop.options,
                                {
                                    // we can use these 2 values to get the other value for the property
                                    isOther: true,
                                    parent: prop.id,
                                    id: null,
                                    name: "other",
                                    slug: `${prop.slug}-other`,
                                    child: false,
                                },
                            ];

                            return Object.assign({}, prop, {
                                label: prop.name,
                                value: prop.id,
                                options,
                            });
                        });

                        return mappedChild;

                        // push the child properties to the properties array after the index
                    }
                });
        },
        getMainCategoriesWithSubCategories() {
            axios
                .get("https://staging.mazaady.com/api/get_all_cats")
                .then(({ data }) => {
                    if (data && data.code !== 200) {
                        this.mainCategoriesWithSubCategories = [
                            {
                                label: "unable to connect to Mazaady Server",
                            },
                        ];
                    }

                    this.mainCategoriesWithSubCategories =
                        data.data.categories.map((category) => {
                            return {
                                label: category.name,
                                value: category.id,
                                children: category.children.map((child) => {
                                    return {
                                        label: child.name,
                                        value: child.id,
                                    };
                                }),
                            };
                        });
                });
        },

        updateSelectedProperties(propData, option) {
            this.selectedProperties[option.id] = propData;
        },

        updateProperties(propData, option) {
            /*
            propData Example:
            {
                id: 55,
                name: "BMW",
                label: "BMW",
                slug: "color-other",
                parent: 2,
                value: 55
                child: true,
            }

            option Example:
            {
                id: 1,
                parent: null,
                name: "Brand",
                slug: "brand",
                label: true,
            }

          1- check if the option has a child
          2- if it has a child, get the child properties
          3- remove other child properties from the properties array
          4- add the new child properties to the properties array after the index of the option
          5- update the selected properties
           */

            if (propData.child) {
                this.getChildProperties(propData.id).then((childProps) => {
                    const index = this.properties.indexOf(option);
                    this.properties.splice(index + 1, 1);
                    this.properties.splice(index + 1, 0, ...childProps);
                });
            }
        },

        mapOptions(options) {
            return [...options].map((option) => {
                return {
                    label: option.name,
                    value: option.id,
                    ...option,
                };
            });
        },

        loadPropertiesForSelectedSubCategory({ value, label }) {
            axios
                .get(`https://staging.mazaady.com/api/properties?cat=${value}`)
                .then(({ data }) => {
                    if (data && data.code == 200) {
                        this.properties = data.data.map((prop) => {
                            const options = [
                                ...prop.options,
                                {
                                    id: null,
                                    name: "other",
                                    slug: `${prop.slug}-other`,
                                    parent: prop.id,
                                    child: false,
                                },
                            ];

                            return Object.assign({}, prop, {
                                label: prop.name,
                                value: prop.id,
                                options,
                            });
                        });
                    }
                });
        },
    },
};
</script>
