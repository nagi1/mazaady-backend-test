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
