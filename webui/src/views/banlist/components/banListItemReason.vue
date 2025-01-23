<template>
    <a-descriptions-item :label="t('page.banlist.banlist.listItem.reason')" :span="12">
        <template v-if="descriptionCount[0] == 1 || descriptionCount[1] == 0">
            <a-typography-text style="margin-bottom: 0" :ellipsis="{ showTooltip: true }">
                {{ descriptionBrief }}
            </a-typography-text>
        </template>
        <template v-else>
            <a-typography-text style="margin-bottom: 0">
                <a-popover title="封禁原因" :position="'tl'">
                    <a-link style="color: #1d2129">{{ descriptionBrief }}</a-link>
                    <a-tag color="blue" bordered>{{ descriptionCount[0] }}条折叠</a-tag>
                    <template #content>
                        <template v-for="i in descriptionList">
                            <p>
                                <template v-if="i.groups.length > 1">
                                    <span>有</span>
                                    <span class="banDescriptionSpan bds-blue">{{ i.groups.length }}</span>
                                    <span>条记录</span>
                                </template>

                                <!-- 避免使用 v-html 实现渲染所以模板复杂了一点 -->
                                <template v-for="j of i.pattern.format.split('|')">
                                    <template v-if="j.includes(',')">
                                        <span class="banDescriptionSpan">
                                            <template v-for="k of j.split(',')">
                                                <template v-if="k in countAll[i.pattern.name]">
                                                    {{ countAll[i.pattern.name][k] }}
                                                </template>
                                                <template v-else>
                                                    {{ k }}
                                                </template>
                                            </template>
                                        </span>
                                    </template>
                                    <template v-else>
                                        <template v-if="j in countAll[i.pattern.name]">
                                            <span class="banDescriptionSpan">
                                                {{ countAll[i.pattern.name][j] }}
                                            </span>
                                        </template>
                                        <template v-else>
                                            {{ j }}
                                        </template>
                                    </template>
                                </template>

                            </p>
                        </template>
                        <p style="color: #9A9A9A;">
                            正在展示 {{ descriptionCount[0] }} 条中的 {{ descriptionCount[1] }} 条，可点击查看原始记录
                        </p>
                    </template>
                </a-popover>
            </a-typography-text>
        </template>
    </a-descriptions-item>
</template>

<style>
.banDescriptionSpan {
    margin-left: .3rem;
    margin-right: .3rem;
    padding-left: .2rem;
    padding-right: .2rem;
    border-radius: .3rem;
    font-weight: bold;
}

.bds-orange {
    background-color: #FF9A2E;
}

.bds-blue {
    background-color: #4080FF;
    color: #FFFFFF;
}
</style>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { onMounted, ref } from 'vue'
import type { Ref } from 'vue'
import { computed } from 'vue'
import patternJSON from "./banListItemReasonPattern.json"

const props = defineProps(['description'])
const { t, d } = useI18n()

interface interfaceDescriptionList {
    groups: { [key: string]: any },
    pattern: {
        name: string,
        pattern: string,
        group_data: string[],
        format: string,
    },
}

const descriptionList: Ref<{ [key: string]: interfaceDescriptionList }> = ref({})
const descriptionCount: Ref<[number, number]> = ref([0, 0])
const descriptionBrief = ref('')

const countAll = computed(() => {
    let result: { [key: string]: { [key: string]: number } } = {}
    for (let pattern_name in descriptionList.value) {
        result[pattern_name] = {}
        for (let meta_name of descriptionList.value[pattern_name].pattern.group_data) {
            if (!(meta_name in result[pattern_name])) {
                result[pattern_name][meta_name] = 0
            }
            for (let current_group of descriptionList.value[pattern_name].groups) {

                let ratio = 1
                if ((meta_name + "_unit") in current_group) {
                    let unit = current_group[meta_name + "_unit"]
                    if (unit == "MB") {
                        ratio = 1 / 1024
                    }
                    if (unit == "TB") {
                        ratio = 1024
                    }
                }

                result[pattern_name][meta_name] += parseFloat(current_group[meta_name]) * ratio
            }
            result[pattern_name][meta_name] = Math.round(result[pattern_name][meta_name])
        }
        if (descriptionList.value[pattern_name].groups) {
            for (let meta_name in descriptionList.value[pattern_name].groups[0]) {
                if (meta_name in result[pattern_name]) {
                    continue
                }
                result[pattern_name][meta_name] = descriptionList.value[pattern_name].groups[0][meta_name]
            }
        }
    }
    return result
})

onMounted(() => {
    for (let current_description_line of props.description.split("\n")) {
        descriptionCount.value[0] += 1
        current_description_line = current_description_line.trim()
        if (current_description_line.endsWith(".txt")) {
            continue
        }
        for (let current_pattern of patternJSON) {
            let matches: RegExpExecArray | null = new RegExp(current_pattern.pattern).exec(current_description_line)
            if (matches === null) {
                continue
            }
            if (matches.groups === undefined) {
                continue
            }
            if (!descriptionBrief.value) {
                descriptionBrief.value = current_description_line
            }
            descriptionCount.value[1] += 1
            if (!(current_pattern.name in descriptionList.value)) {
                descriptionList.value[current_pattern.name] = { groups: [], pattern: current_pattern }
            }
            descriptionList.value[current_pattern.name].groups.push(matches.groups)
            break
        }
    }
    if (!descriptionBrief.value) {
        descriptionBrief.value = props.description
    }
})
</script>