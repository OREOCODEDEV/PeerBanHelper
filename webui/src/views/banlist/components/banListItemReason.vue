<template>
    <a-descriptions-item :label="t('page.banlist.banlist.listItem.reason')" :span="12">
        <div>
            <a-typography-text style="margin-bottom: 0">
                <a-popover title="Title" :position="'tl'">
                    <span>{{ descriptionBrief }}</span>
                    <template #content>
                        <template v-for="i of descriptionList">
                            <p>
                                <template v-if="i.count > 1">
                                    <span>有</span>
                                    <span class="banDescriptionSpan bds-orange">{{ i.count }}</span>
                                    <span>条记录共</span>
                                </template>
                                <template v-for="j of i.format.split('|')">
                                    <template v-for="k of j.split(',')">
                                        <template v-if="k in i.groups">
                                            {{ i.groups[k] }}
                                        </template>
                                        <template v-else>
                                            {{ k }}
                                        </template>
                                    </template>
                                </template>
                            </p>
                        </template>
                        <p>
                            正在展示 {{ descriptionCount[0] }} 条中的 {{ descriptionCount[1] }} 条，可点击查看原始记录
                        </p>
                    </template>
                </a-popover>
            </a-typography-text>
        </div>
    </a-descriptions-item>
</template>

<style>
.banDescriptionSpan {
    margin-left: .3rem;
    margin-right: .3rem;
    padding-left: .2rem;
    padding-right: .2rem;
    border-radius: .3rem;
    color: #FFFFFF;
}

.bds-orange {
    background-color: #FF9A2E;
}
</style>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { onMounted, ref } from 'vue'
import type { Ref } from 'vue'
import patternJSON from "./banListItemReasonPattern.json"

const props = defineProps(['description'])
const { t, d } = useI18n()

interface interfaceDescriptionList {
    name: string,
    groups: { [key: string]: any },
    count: number,
    format: string
    // [key: string]: any
}

const descriptionList: Ref<interfaceDescriptionList[]> = ref([])
const descriptionCount: Ref<[number, number]> = ref([0, 0])
const descriptionBrief = ref('')

onMounted(() => {
    for (let current_description_line of props.description.split("\n")) {
        descriptionCount.value[0] += 1
        current_description_line = current_description_line.trim()
        if (current_description_line.endsWith(".txt")) {
            continue
        }
        let __debug_match_flag = false
        for (let current_pattern of patternJSON) {
            let matches: RegExpExecArray | null = new RegExp(current_pattern.pattern).exec(current_description_line)
            if (matches === null) {
                continue
            }
            if (matches.groups === undefined) {
                continue
            }
            descriptionCount.value[1] += 1
            __debug_match_flag = true
            if (!current_pattern.group_data) {
                descriptionList.value.push({ name: current_pattern.name, groups: matches.groups, count: 1, format: current_pattern.format })
                continue
            }
            let displayFlag = false
            for (let i of descriptionList.value) {
                if (i.name != current_pattern.name) {
                    continue
                }
                displayFlag = true
                i.count += 1
                for (let j of current_pattern.group_data) {
                    if (typeof i.groups[j] === 'string') {
                        i.groups[j] = parseFloat(i.groups[j])
                    }
                    i.groups[j] += parseFloat(matches.groups[j])
                }
            }
            if (!displayFlag) {
                descriptionList.value.push({ name: current_pattern.name, groups: matches.groups, count: 1, format: current_pattern.format })
            }
            break
        }
        if (!__debug_match_flag) {
            console.warn("Unmatched description:", current_description_line)
        }
    }
    if (!descriptionBrief.value) {
        descriptionBrief.value = props.description
        // debugger
    }
})
</script>