<template>
    <el-input
        :data-component="dataComponent"
        v-model="search"
        @input="onInput"
        :placeholder="$t(placeholder)"
        :readonly="readonly"
    >
        <template #prefix>
            <slot name="prefix" />
        </template>
        <template #suffix>
            <div class="shortcut d-flex">
                <slot name="suffix">
                    <magnify />
                </slot>
            </div>
        </template>
    </el-input>
</template>
<script>
    import {debounce} from "throttle-debounce";
    import Magnify from "vue-material-design-icons/Magnify.vue";
    import BaseComponents from "../BaseComponents.vue"

    export default {
        extends: BaseComponents,
        emits: ["search"],
        components: {Magnify},
        searchDebounce: undefined,
        created() {
            this.init();
        },
        props: {
            router: {
                type: Boolean,
                default: true
            },
            placeholder: {
                type: String,
                required: false,
                default: "search"
            },
            readonly: {
                type: Boolean
            }
        },
        watch: {
            $route(newValue, oldValue) {
                if (oldValue.name === newValue.name) {
                    this.init()
                }
            }
        },
        data() {
            return {
                search: ""
            };
        },
        methods: {
            init() {
                if (this.$route.query.q && this.router) {
                    this.search = this.$route.query.q;
                }
                this.searchDebounce = debounce(300, () => {
                    this.$emit("search", this.search);

                    if (this.router) {
                        const query = {...this.$route.query, q: this.search, page: 1};
                        if (!this.search) {
                            delete query.q;
                        }
                        this.$router.push({query});
                    }
                });
            },
            onInput() {
                this.searchDebounce();
            },
        },
        unmounted() {
            this.searchDebounce && this.searchDebounce.cancel();
        }
    };
</script>
<style lang="scss" scoped>
    .shortcut {
        font-size: 0.75rem;
        line-height: 1.25rem;
        gap: calc(var(--spacer) / 4);
    }

    .el-input {
        :deep(.el-input__prefix), :deep(input)::placeholder {
            color: var(--bs-body-color);
        }
    }
</style>