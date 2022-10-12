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
                        (selectedOption) => {
                            updateProperties(selectedOption, prop);
                            updateSelectedProperties(selectedOption, prop);
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
            propertiesMap: [],

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
        properties: {
            handler: function (newVal, oldVal) {
                this.$emit("properties", this.getAllSelectedProperties);
            },
            deep: true,
        },
    },
    computed: {
        getAllSelectedProperties() {
            // merge selectedProperties with filteredProperties by id
            const mappedProperties = this.properties
                .filter((prop) => prop)
                .map((prop) => {
                    if (this.selectedProperties.hasOwnProperty(prop.id)) {
                        return this.selectedProperties[prop.id];
                    }
                    return prop;
                })
                .map((prop) => {
                    console.log(prop);
                    if (
                        this.selectedProperties
                            .filter((prop) => prop)
                            .findIndex((p) => p.id === prop.id) !== -1
                    ) {
                        return {
                            id: prop.id,
                            key: prop.isOther
                                ? prop.originalName
                                : this.propertiesMap[prop.id],
                            value: prop.isOther ? prop.value : prop.name,
                        };
                    }

                    return {
                        id: prop.id,
                        key: prop.name,
                        value: null,
                    };
                });

            return {
                mainCategory: this.mainCategory,
                subCategory: this.subCategory,
                properties: mappedProperties,
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
                                    originalName: prop.name,
                                    parent: prop.id,
                                    id: prop.id,
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

        updateSelectedProperties(selectedOption, option) {
            this.selectedProperties[option.id] = selectedOption;
            this.propertiesMap[selectedOption.id] = option.label;
        },

        getPropertiesToRemove(option, childrenToRemove) {
            const children = this.properties.filter((prop) => {
                return prop.parent && prop.parent === option.id;
            });
            children.push(
                ...this.selectedProperties.filter((prop) => {
                    return prop.parent && prop.parent === option.id;
                })
            );

            if (children.length === 0) {
                return;
            }

            childrenToRemove.push(...children);

            children.forEach((child) => {
                return this.getPropertiesToRemove(child, childrenToRemove);
            });
        },

        updateProperties(selectedOption, option) {
            /*
            selectedOption Example:
            {
                id: 55,
                name: "BMW",
                label: "BMW",
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
          3- remove previous selected child properties from the properties array
          4- add the new child properties to the properties array after the index of the option
          5- update the selected properties
           */

            if (selectedOption.child) {
                const previousSelectedValue =
                    this.selectedProperties[option.id];

                this.getChildProperties(selectedOption.id).then(
                    (childProps) => {
                        // get the children of the previousSelectedValue property
                        if (previousSelectedValue) {
                            /*
                               1- get the children of the previousSelectedValue property
                               2- get the children of the children of the previousSelectedValue property until there is no children
                                 3- remove the children from the properties array
                            */
                            const childrenToRemove = [];

                            this.getPropertiesToRemove(
                                previousSelectedValue,
                                childrenToRemove
                            );
                            // remove the childrenTree from the properties array
                            this.properties = this.properties.filter(
                                (prop) => !childrenToRemove.includes(prop)
                            );
                        }

                        const index = this.properties.indexOf(option);
                        this.properties.splice(index + 1, 0, ...childProps);
                    }
                );
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
                                    id: prop.id,
                                    isOther: true,
                                    name: "other",
                                    originalName: prop.name,
                                    value: null,
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
