<template>
    <a-descriptions-item :label="t('page.banlist.banlist.listItem.reason')" :span="12">
        <div>
            <a-typography-text style="margin-bottom: 0">
                <a-popover title="Title" :position="'tl'">
                    <span>{{ descriptionBrief }}</span>
                    <template #content>
                        <p>Here is the text content</p>
                        <p>Here is the text content</p>
                    </template>
                </a-popover>
            </a-typography-text>
        </div>
    </a-descriptions-item>
</template>
<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { onMounted, ref } from 'vue'
import type { Ref } from 'vue'

const props = defineProps(['description'])
const { t, d } = useI18n()
const descriptionList: Ref<banDescription[]> = ref([])
const rawDescriptionCount = ref(0)
const descriptionBrief = ref('')

class banDescription {
    constructor() {
    }
}

class banDescriptionFakeBandwidth extends banDescription {
    static pattern = new RegExp("^\\[AutoGen\\] 从大小为 (?<source_size>[\\d]+) (?<source_unit>.+) 的种子上下载了 (?<download_size>[\\d]+) (?<download_unit>.+) 的数据。下载比\\(100%=完整下载1次种子的大小\\)： (?<download_ratio>[\\d]+[\\.]?[\\d]+)%")
    constructor(
        public source_size: string,
        public source_unit: string,
        public download_size: string,
        public download_unit: string,
        public download_ratio: string
    ) {
        super()
    }
    static handleMatch(data) {
        let groups = data.groups
        return new banDescriptionFakeBandwidth(
            groups.source_size,
            groups.source_unit,
            groups.download_size,
            groups.download_unit,
            groups.download_ratio,
        )
    }
}

class banDescriptionReported extends banDescription {
    static pattern = new RegExp("^\\[AutoGen\\] 被 (?<report_count>[\\d]+) 个用户应用程序上报为不信任的 IP 地址")
    constructor(public report_count: string) {
        super()
    }
    static handleMatch(data) {
        return new banDescriptionReported(data.groups.report_count)
    }
}

class banDescriptionIPRange extends banDescription {
    static pattern = new RegExp("^.+,? \\d+\\.\\d+\\.\\d+\\.\\d+-\\d+\\.\\d+\\.\\d+\\.\\d+")
    constructor(public report_count: string) {
        super()
    }
    static handleMatch(data) {
        return null
    }
}

class banDescriptionFakeUA extends banDescription {
    static pattern = new RegExp("")
    constructor(public peerId: string, public peerName: string) {
        super()
    }
    // static handleMatch(data) {
    // }
}

onMounted(() => {
    for (let i of props.description.split("\n")) {
        i = i.trim()
        rawDescriptionCount.value += 1
        if (i.endsWith(".txt")) {
            continue
        }
        let matchFlag = false
        for (let j of [banDescriptionFakeBandwidth, banDescriptionReported, banDescriptionIPRange]) {
            let matches = j.pattern.exec(i)
            if (matches === null) {
                continue
            }
            matchFlag = true
            let instince = j.handleMatch(matches)
            if (!instince) {
                continue
            }
            descriptionList.value.push(instince)
            break
        }
        if (!matchFlag) {
            console.warn("Unmatched description:", i)
        }
    }
    if (!descriptionBrief.value) {
        descriptionBrief.value = props.description
        // debugger
    }
})
</script>