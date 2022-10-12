<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Mazaady Portal Web Task</div>

                    <div class="card-body">
                        <form action="#" method="post">
                            <category-wrapper
                                @properties="handelUpdatingProperties"
                            />
                            <div class="row mt-5">
                                <div class="form-group">
                                    <button
                                        @click="submitForm"
                                        class="btn btn-primary"
                                        type="submit"
                                    >
                                        Search
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-header">Results</div>

                    <div class="card-body">
                        <table class="table">
                            <thead>
                                <tr>
                                    <th scope="col">Key</th>
                                    <th scope="col">Value</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr>
                                    <th scope="row">Main Category</th>
                                    <td
                                        v-text="
                                            properties.mainCategory?.label ??
                                            'Not Selected'
                                        "
                                    ></td>
                                </tr>
                                <tr>
                                    <th scope="row">Sub Category</th>
                                    <td
                                        v-text="
                                            properties.subCategory?.label ??
                                            'Not Selected'
                                        "
                                    ></td>
                                </tr>
                                <tr
                                    v-for="prop in properties.properties"
                                    :key="prop.id"
                                >
                                    <th scope="row" v-text="prop.key"></th>
                                    <td
                                        v-text="prop.value ?? 'Not Selected'"
                                    ></td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            properties: [],
        };
    },
    methods: {
        handelUpdatingProperties(properties) {
            this.properties = properties;
        },

        submitForm(e) {
            e.preventDefault();
            console.log(this.properties);
        },
    },
};
</script>
