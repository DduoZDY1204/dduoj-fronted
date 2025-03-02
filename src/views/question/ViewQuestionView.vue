<template>
    <div id="viewQuestionView">
        <a-row :gutter="[24, 24]">
            <a-col :md="12" :xs="24">
                <a-tabs default-active-key="question">
                    <a-tab-pane key="question" title="题目">
                        <a-card v-if="question" :title="question.title">
                            <a-descriptions
                                    title="判题条件"
                                    :column="{ xs: 1, md: 2, lg: 3 }"
                            >
                                <a-descriptions-item label="时间限制">
                                    {{ question.judgeConfig.timeLimit ?? 0 }}
                                </a-descriptions-item>
                                <a-descriptions-item label="内存限制">
                                    {{ question.judgeConfig.memoryLimit ?? 0 }}
                                </a-descriptions-item>
                                <a-descriptions-item label="堆栈限制">
                                    {{ question.judgeConfig.stackLimit ?? 0 }}
                                </a-descriptions-item>
                            </a-descriptions>

                            <MdViewer :value="question.content || ''"/>

                            <template #extra>
                                <a-space wrap>
                                    <a-tag
                                            v-for="(tag, index) of question.tags"
                                            :key="index"
                                            color="green"
                                    >{{ tag }}
                                    </a-tag>
                                </a-space>
                            </template>
                        </a-card>
                    </a-tab-pane>
                    <a-tab-pane key="comment" title="评论" disabled> 评论区</a-tab-pane>
                    <a-tab-pane key="answer" title="答案"> 暂时无法查看答案</a-tab-pane>
                </a-tabs>
            </a-col>
            <a-col :md="12" :xs="24">
                <a-form
                  :model="form"
                  layout="inline">
                    <a-form-item
                            field="language"
                            label="编程语言"
                            style="min-width: 240px"
                    >
                        <a-select
                                v-model="form.language"
                                :style="{ width: '320px' }"
                                placeholder="选择编程语言"
                        >
                            <a-option>java</a-option>
                            <a-option>(目前未开放其他语言判题逻辑 只适配Java)</a-option>
                        </a-select>
                    </a-form-item>
                </a-form>

                <CodeEditor
                    :value="form.code as string"
                    :language="form.language"
                    :handle-change="changeCode"
                    style="text-align: justify"
                />

                <a-divider size="0"/>

                <div>
                    <a-button
                      type="primary"
                      style="min-width: 200px"
                      @click="doSubmit"
                      :loading="isButtonLoading"
                      :disabled="isButtonDisabled"
                    >
                        {{ isButtonDisabled ? countdown + '秒后再试' : '提交代码' }}
                    </a-button>
                </div>

                <div>
                    <br>
                </div>

                <div>
                    <a-button
                      type="primary"
                      @click="toQuestionSubmitView"
                    >
                        点击我跳转到题目提交列表
                    </a-button>
                </div>

            </a-col>
        </a-row>
    </div>
</template>

<script setup lang="ts">
import {onMounted, ref, watchEffect, withDefaults, defineProps} from "vue";
import message from "@arco-design/web-vue/es/message";
import CodeEditor from "@/components/CodeEditor.vue";
import MdViewer from "@/components/MdViewer.vue";

import {
    QuestionControllerService,
    QuestionSubmitAddRequest,
    QuestionVO,
} from "../../../generated";
import { useRouter } from "vue-router";

interface Props {
    id: string;
}

const props = withDefaults(defineProps<Props>(), {
    id: () => "",
});

const question = ref<QuestionVO>();

// 按钮的状态
const isButtonLoading = ref(false);
const isButtonDisabled = ref(false);
// 时间
const countdown = ref(0);
const router = useRouter();

const loadData = async () => {
    const res = await QuestionControllerService.getQuestionVoByIdUsingGet(
        props.id as any
    );
    if (res.code === 0) {
        question.value = res.data;
    } else {
        message.error("加载失败，" + res.message);
    }
};

const form = ref<QuestionSubmitAddRequest>({
    language: "java",
    code: "",
});

/**
 * 提交代码
 */
const doSubmit = async () => {
    handleButtonClick();

    console.log("问题的id"+question.value?.id);

    if (!question.value?.id) {
        return;
    }

    console.log("进行了提交");

    const res = await QuestionControllerService.doQuestionSubmitUsingPost({
        ...form.value,
        questionId: question.value.id,
    });

    console.log(res.code);

    if (res.code === 0) {
        message.success("提交成功");
        // 不用进行页面的跳转
        // router.push({
        //     path: "/QuestionSubmitView",
        //     replace: true,
        // });
    } else {
        message.error("提交失败," + res.message);
    }
};

const toQuestionSubmitView = async () =>{
    // 跳转到题目提交列表
    router.push({
        path: "/QuestionSubmitView",
        replace: true,
    });
}

/**
 * 页面加载时，请求数据
 */
onMounted(() => {
    loadData();
});

// 处理按钮点击事件
const handleButtonClick = () => {
    // 禁用按钮并开始倒计时
    isButtonDisabled.value = true;
    countdown.value = 10;
    const interval = setInterval(() => {
        countdown.value -= 1;
        if (countdown.value <= 0) {
            clearInterval(interval);
            isButtonDisabled.value = false;
        }
    }, 1000);
};

const changeCode = (value: string) => {
    form.value.code = value;
};
</script>

<style>
#viewQuestionView {
    max-width: 1400px;
    margin: 0 auto;
}

#viewQuestionView .arco-space-horizontal .arco-space-item {
    margin-bottom: 0 !important;
}
</style>
