# Fine-tuning (파인튜닝)

## 1. 한 줄 요약

사전 학습된 대형 모델을 특정 도메인이나 작업에 맞게 추가 학습시켜 성능을 최적화하는 기법입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

Fine-tuning은 **iOS 기본 앱을 커스터마이징**하는 것과 비슷합니다.

```
처음부터 개발                     Fine-tuning
┌─────────────────┐              ┌────────────────────────────┐
│                 │              │                            │
│  앱을 0부터     │              │  iOS 기본 메일 앱을        │
│  완전히 새로    │              │  회사 메일 시스템에 맞게   │
│  만들기         │              │  설정만 변경               │
│                 │              │                            │
│  - 수년 소요    │              │  - 몇 시간 소요            │
│  - 엄청난 비용  │              │  - 적은 비용               │
│  - 전문가 필요  │              │  - 설정만으로 가능         │
│                 │              │                            │
└─────────────────┘              └────────────────────────────┘
```

### Fine-tuning의 핵심 개념

```
┌─────────────────────────────────────────────────────────────────┐
│                    Fine-tuning 과정                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Pre-trained Model          +  Custom Data   =   Fine-tuned   │
│   (사전학습 모델)                (우리 데이터)     (최적화 모델)│
│                                                                 │
│   ┌─────────────────┐      ┌─────────────┐    ┌─────────────┐  │
│   │ GPT, Claude 등  │  +   │ 고객상담 로그│ =  │ 우리 회사   │  │
│   │ 일반 지식 보유  │      │ 제품 메뉴얼  │    │ 전문 챗봇   │  │
│   │                 │      │ FAQ 데이터   │    │             │  │
│   └─────────────────┘      └─────────────┘    └─────────────┘  │
│                                                                 │
│   "영어, 수학, 과학을        "우리 회사          "우리 회사에  │
│    다 배운 졸업생"           제품에 대해 교육"    특화된 전문가"│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 언제 Fine-tuning이 필요한가?

```
RAG로 충분한 경우                    Fine-tuning이 필요한 경우
┌─────────────────────────┐         ┌─────────────────────────┐
│ ✓ 최신 정보가 필요      │         │ ✓ 특정 말투/스타일 필요 │
│ ✓ 출처가 중요           │         │ ✓ 전문 용어 이해 필요   │
│ ✓ 자주 업데이트됨       │         │ ✓ 복잡한 추론 필요     │
│ ✓ 정보 검색이 주목적    │         │ ✓ 일관된 형식 출력     │
│                         │         │ ✓ 응답 속도가 중요     │
│ 예: FAQ 챗봇, 문서 검색 │         │ 예: 법률 분석, 의료 진단│
└─────────────────────────┘         └─────────────────────────┘
```

---

## 3. 구조 다이어그램

### Fine-tuning 방법론 비교

```
┌─────────────────────────────────────────────────────────────────┐
│                    Fine-tuning 방법론                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Full Fine-tuning (전체 미세조정)                           │
│     ┌─────────────────────────────────────────────────────┐    │
│     │  모든 파라미터 업데이트                              │    │
│     │  ████████████████████████████████████ (100%)        │    │
│     │  - 최고 성능, 최대 비용                              │    │
│     │  - GPU 메모리 많이 필요 (7B 모델에 ~60GB VRAM)       │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
│  2. LoRA (Low-Rank Adaptation) - 가장 인기                     │
│     ┌─────────────────────────────────────────────────────┐    │
│     │  기존 모델 동결 + 작은 어댑터 추가                   │    │
│     │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░████ (0.1~1%)    │    │
│     │  - 효율적, 여러 버전 관리 용이                       │    │
│     │  - 7B 모델에 ~16GB VRAM으로 가능                     │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
│  3. QLoRA (Quantized LoRA)                                     │
│     ┌─────────────────────────────────────────────────────┐    │
│     │  4bit 양자화 + LoRA                                  │    │
│     │  메모리 사용량 4배 감소                              │    │
│     │  - 소비자용 GPU에서도 가능 (RTX 4090으로 7B 학습)    │    │
│     │  - 약간의 성능 저하 (1-3%)                           │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
│  4. Prompt Tuning / Prefix Tuning                              │
│     ┌─────────────────────────────────────────────────────┐    │
│     │  학습 가능한 프롬프트 토큰만 추가                    │    │
│     │  모델 자체는 변경 없음                               │    │
│     │  - 매우 가벼움 (수백 KB)                             │    │
│     │  - 성능은 제한적                                     │    │
│     └─────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### LoRA 작동 원리

```
┌─────────────────────────────────────────────────────────────────┐
│                      LoRA 원리                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  기존 가중치 행렬 W (동결)                                      │
│  ┌─────────────────┐                                           │
│  │  d x d 행렬     │  (예: 4096 x 4096 = 16M 파라미터)         │
│  │  (매우 큼)      │                                           │
│  └─────────────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  LoRA: 저랭크 분해로 근사                                       │
│  ┌───────┐     ┌───────┐                                       │
│  │ A     │  x  │   B   │  = ΔW                                 │
│  │ d x r │     │ r x d │  (r << d, 예: r=8)                    │
│  └───────┘     └───────┘                                       │
│                                                                 │
│  학습 파라미터: 2 x d x r = 2 x 4096 x 8 = 65K (0.4%!)         │
│                                                                 │
│  추론 시: Y = X(W + ΔW) = XW + X·A·B                           │
│                                                                 │
│  장점:                                                          │
│  - 원본 모델 그대로 유지                                        │
│  - 여러 LoRA 어댑터를 동적으로 교체 가능                        │
│  - 작은 파일 크기 (수십 MB vs 수십 GB)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Fine-tuning 파이프라인

```
┌─────────────────────────────────────────────────────────────────┐
│                  Fine-tuning 전체 파이프라인                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 데이터 준비                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                     │
│  │ 원본    │───▶│ 정제    │───▶│ 포맷팅  │                     │
│  │ 데이터  │    │ 필터링  │    │ JSONL   │                     │
│  └─────────┘    └─────────┘    └─────────┘                     │
│                                      │                          │
│  2. 학습                             ▼                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Base Model + Training Data ───▶ Fine-tuned Model       │   │
│  │                                                         │   │
│  │  Hyperparameters:                                       │   │
│  │  - epochs: 3-5                                          │   │
│  │  - batch_size: 4-16                                     │   │
│  │  - learning_rate: 1e-5 ~ 5e-5                          │   │
│  │  - LoRA rank: 8-64                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                      │                          │
│  3. 평가                             ▼                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Validation Set으로 성능 측정                           │   │
│  │  - Perplexity (낮을수록 좋음)                          │   │
│  │  - Task-specific metrics                                │   │
│  │  - Human evaluation                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                      │                          │
│  4. 배포                             ▼                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - 모델 서빙 (vLLM, TGI, Ollama)                        │   │
│  │  - API 엔드포인트                                       │   │
│  │  - A/B 테스트                                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2024-2025년 Fine-tuning 트렌드

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Fine-tuning 최신 트렌드 (2024-2025)                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 관리형 Fine-tuning 서비스                                          │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • OpenAI Fine-tuning: GPT-4o-mini, GPT-4o 지원             │   │
│     │  • Claude Fine-tuning: Amazon Bedrock 통해 제공              │   │
│     │  • Google Vertex AI: Gemini 모델 튜닝                        │   │
│     │  • Together.ai: 다양한 오픈소스 모델 Fine-tuning            │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. 효율적 Fine-tuning 기법                                            │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • DoRA: LoRA 개선, 방향과 크기 분리 학습                   │   │
│     │  • LongLoRA: 긴 컨텍스트 효율적 학습                        │   │
│     │  • AdaLoRA: 중요도 기반 동적 랭크 조절                      │   │
│     │  • ReLoRA: 점진적 랭크 증가                                  │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. Preference Optimization (선호도 최적화)                            │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • DPO (Direct Preference Optimization)                     │   │
│     │    - RLHF 대체, 더 간단하고 안정적                          │   │
│     │  • ORPO: SFT와 선호도 학습 통합                              │   │
│     │  • IPO: Identity Preference Optimization                    │   │
│     │  • KTO: Kahneman-Tversky Optimization                       │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  4. 합성 데이터 Fine-tuning                                            │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 강력한 모델(GPT-4o)로 학습 데이터 생성                   │   │
│     │  • Self-instruct, Evol-Instruct 기법                        │   │
│     │  • 데이터 부족 문제 해결                                     │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 1. OpenAI Fine-tuning

```python
# OpenAI Fine-tuning API 사용 예시
from openai import OpenAI
import json

client = OpenAI()

# 상수 정의
MIN_EXAMPLES = 10
RECOMMENDED_EXAMPLES = 100

# 1. 학습 데이터 준비 (JSONL 형식)
training_data = [
    {
        "messages": [
            {"role": "system", "content": "당신은 테크 회사의 고객 지원 전문가입니다. 친절하고 정확하게 답변합니다."},
            {"role": "user", "content": "결제가 안 돼요"},
            {"role": "assistant", "content": "불편을 드려 죄송합니다. 결제 오류 해결을 도와드리겠습니다.\n\n1. 어떤 결제 수단을 사용하셨나요? (신용카드/체크카드/간편결제)\n2. 오류 메시지가 표시되었다면 알려주세요.\n\n일반적인 해결 방법:\n- 카드 한도 확인\n- 다른 결제 수단 시도\n- 앱 재시작 후 재시도\n\n추가 정보를 주시면 더 정확히 도움드릴 수 있습니다."}
        ]
    },
    {
        "messages": [
            {"role": "system", "content": "당신은 테크 회사의 고객 지원 전문가입니다. 친절하고 정확하게 답변합니다."},
            {"role": "user", "content": "앱이 자꾸 꺼져요"},
            {"role": "assistant", "content": "앱 강제 종료 문제를 겪고 계시군요. 다음 순서로 해결해보세요:\n\n**즉시 해결 시도:**\n1. 앱 버전이 최신인지 확인 (앱스토어에서 업데이트)\n2. 기기 재시작\n3. 앱 캐시 삭제 (설정 > 앱 > 저장공간 > 캐시 삭제)\n\n**추가 확인:**\n- 기기 저장 공간이 충분한가요? (최소 1GB 권장)\n- 다른 앱도 종료되나요?\n\n위 방법으로 해결되지 않으면 앱 삭제 후 재설치를 권장드립니다."}
        ]
    },
    # ... 수백~수천 개의 예시
]

# JSONL 파일로 저장
def save_training_data(data: list, filepath: str):
    with open(filepath, "w", encoding="utf-8") as f:
        for item in data:
            f.write(json.dumps(item, ensure_ascii=False) + "\n")

save_training_data(training_data, "training_data.jsonl")

# 2. 파일 업로드
file = client.files.create(
    file=open("training_data.jsonl", "rb"),
    purpose="fine-tune"
)
print(f"File ID: {file.id}")

# 3. Fine-tuning 작업 생성
job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-4o-mini-2024-07-18",  # 베이스 모델
    hyperparameters={
        "n_epochs": 3,
        "batch_size": "auto",  # 자동 최적화
        "learning_rate_multiplier": "auto"
    },
    suffix="customer-support"  # 모델 이름에 추가될 접미사
)
print(f"Job ID: {job.id}")

# 4. 학습 상태 확인
def check_training_status(job_id: str):
    job = client.fine_tuning.jobs.retrieve(job_id)
    print(f"Status: {job.status}")
    print(f"Trained tokens: {job.trained_tokens}")

    if job.status == "succeeded":
        print(f"Fine-tuned model: {job.fine_tuned_model}")
    elif job.status == "failed":
        print(f"Error: {job.error}")

    return job

# 학습 이벤트 조회
events = client.fine_tuning.jobs.list_events(fine_tuning_job_id=job.id, limit=10)
for event in events.data:
    print(f"{event.created_at}: {event.message}")

# 5. Fine-tuned 모델 사용
def chat_with_fine_tuned_model(model_id: str, user_message: str) -> str:
    response = client.chat.completions.create(
        model=model_id,  # "ft:gpt-4o-mini-2024-07-18:org:customer-support:abc123"
        messages=[
            {"role": "system", "content": "당신은 테크 회사의 고객 지원 전문가입니다."},
            {"role": "user", "content": user_message}
        ],
        temperature=0.7,
        max_tokens=500
    )
    return response.choices[0].message.content

# 테스트
response = chat_with_fine_tuned_model(
    "ft:gpt-4o-mini-2024-07-18:org:customer-support:abc123",
    "환불 받고 싶어요"
)
print(response)
```

### 2. 오픈소스 모델 Fine-tuning (LoRA)

```python
# Hugging Face + PEFT로 Llama Fine-tuning
import torch
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    TrainingArguments,
    Trainer,
    BitsAndBytesConfig
)
from peft import LoraConfig, get_peft_model, TaskType, prepare_model_for_kbit_training
from datasets import load_dataset

# 상수 정의
MODEL_NAME = "meta-llama/Llama-3.1-8B-Instruct"
OUTPUT_DIR = "./llama-support-lora"
LORA_RANK = 16
LORA_ALPHA = 32
LEARNING_RATE = 2e-4
NUM_EPOCHS = 3
BATCH_SIZE = 4

# 1. 4bit 양자화 설정 (QLoRA)
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True
)

# 2. 모델 및 토크나이저 로드
model = AutoModelForCausalLM.from_pretrained(
    MODEL_NAME,
    quantization_config=bnb_config,
    device_map="auto",
    trust_remote_code=True
)
tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
tokenizer.pad_token = tokenizer.eos_token

# 3. 양자화된 모델 준비
model = prepare_model_for_kbit_training(model)

# 4. LoRA 설정
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=LORA_RANK,                    # LoRA rank
    lora_alpha=LORA_ALPHA,          # 스케일링 파라미터
    lora_dropout=0.1,               # 드롭아웃
    target_modules=[                # 적용할 레이어
        "q_proj", "k_proj", "v_proj", "o_proj",
        "gate_proj", "up_proj", "down_proj"
    ],
    bias="none"
)

# 5. LoRA 적용
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# 출력: trainable params: 13,631,488 || all params: 8,043,720,704 || trainable%: 0.17%

# 6. 데이터셋 로드 및 전처리
def format_instruction(example):
    """Llama 3 instruction 형식으로 변환"""
    text = f"""<|begin_of_text|><|start_header_id|>system<|end_header_id|>

당신은 친절한 고객 지원 전문가입니다.<|eot_id|><|start_header_id|>user<|end_header_id|>

{example['question']}<|eot_id|><|start_header_id|>assistant<|end_header_id|>

{example['answer']}<|eot_id|>"""
    return {"text": text}

# 데이터셋 로드 (직접 만든 JSONL 또는 Hugging Face 데이터셋)
dataset = load_dataset("json", data_files="training_data.json")
dataset = dataset.map(format_instruction)

# 토크나이징
def tokenize(example):
    return tokenizer(
        example["text"],
        truncation=True,
        max_length=2048,
        padding="max_length"
    )

tokenized_dataset = dataset.map(tokenize, batched=True)

# 7. 학습 설정
training_args = TrainingArguments(
    output_dir=OUTPUT_DIR,
    num_train_epochs=NUM_EPOCHS,
    per_device_train_batch_size=BATCH_SIZE,
    gradient_accumulation_steps=4,
    learning_rate=LEARNING_RATE,
    warmup_ratio=0.1,
    logging_steps=10,
    save_steps=500,
    save_total_limit=3,
    fp16=True,
    optim="paged_adamw_32bit",
    report_to="wandb",  # Weights & Biases 로깅
)

# 8. 학습 실행
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset["train"],
    tokenizer=tokenizer,
)

trainer.train()

# 9. LoRA 어댑터 저장
model.save_pretrained(OUTPUT_DIR)
tokenizer.save_pretrained(OUTPUT_DIR)

print(f"LoRA adapter saved to {OUTPUT_DIR}")
```

### 3. DPO (Direct Preference Optimization) 구현

```python
# DPO로 선호도 학습
from trl import DPOTrainer, DPOConfig
from transformers import AutoModelForCausalLM, AutoTokenizer
from datasets import load_dataset

# DPO 데이터셋 형식
# {
#   "prompt": "질문",
#   "chosen": "선호되는 답변",
#   "rejected": "선호되지 않는 답변"
# }

dpo_data = [
    {
        "prompt": "환불 정책이 어떻게 되나요?",
        "chosen": "구매 후 7일 이내 미개봉 상품은 전액 환불 가능합니다. 개봉 상품은 하자가 있는 경우에만 교환/환불됩니다. 환불 요청은 고객센터 또는 마이페이지에서 신청하실 수 있습니다.",
        "rejected": "환불은 7일 이내에 가능합니다."
    },
    {
        "prompt": "배송은 얼마나 걸리나요?",
        "chosen": "일반 배송은 결제 완료 후 2-3일 내 도착하며, 도서 산간 지역은 1-2일 추가 소요될 수 있습니다. 오후 2시 이전 주문 건은 당일 출고됩니다. 실시간 배송 조회는 마이페이지에서 확인 가능합니다.",
        "rejected": "보통 2-3일 걸려요."
    },
]

# DPO 설정
dpo_config = DPOConfig(
    output_dir="./dpo-model",
    num_train_epochs=1,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=4,
    learning_rate=5e-7,
    beta=0.1,  # DPO의 핵심 하이퍼파라미터
    max_length=1024,
    max_prompt_length=512,
)

# 모델 로드 (SFT로 사전 학습된 모델)
model = AutoModelForCausalLM.from_pretrained("./sft-model")
ref_model = AutoModelForCausalLM.from_pretrained("./sft-model")  # 참조 모델
tokenizer = AutoTokenizer.from_pretrained("./sft-model")

# DPO 트레이너
trainer = DPOTrainer(
    model=model,
    ref_model=ref_model,
    args=dpo_config,
    train_dataset=dpo_dataset,
    tokenizer=tokenizer,
)

trainer.train()
```

### 4. Fine-tuned 모델 서빙

```python
# vLLM으로 Fine-tuned 모델 서빙
from vllm import LLM, SamplingParams
from vllm.lora.request import LoRARequest

# 베이스 모델 + LoRA 어댑터 로드
llm = LLM(
    model="meta-llama/Llama-3.1-8B-Instruct",
    enable_lora=True,
    max_lora_rank=32,
    tensor_parallel_size=1,
    gpu_memory_utilization=0.9
)

# 여러 LoRA 어댑터 등록
lora_adapters = {
    "customer-support": "./llama-support-lora",
    "legal-assistant": "./llama-legal-lora",
    "code-reviewer": "./llama-code-lora"
}

# 추론
sampling_params = SamplingParams(
    temperature=0.7,
    max_tokens=512,
    top_p=0.9
)

def generate_with_lora(prompt: str, adapter_name: str) -> str:
    """특정 LoRA 어댑터로 생성"""
    adapter_path = lora_adapters.get(adapter_name)
    if not adapter_path:
        raise ValueError(f"Unknown adapter: {adapter_name}")

    lora_request = LoRARequest(
        lora_name=adapter_name,
        lora_int_id=hash(adapter_name) % 1000,  # 고유 ID
        lora_local_path=adapter_path
    )

    outputs = llm.generate(
        prompts=[prompt],
        sampling_params=sampling_params,
        lora_request=lora_request
    )

    return outputs[0].outputs[0].text

# 사용 예시: 용도별 다른 어댑터
support_response = generate_with_lora(
    "고객: 결제가 안 돼요",
    adapter_name="customer-support"
)

legal_response = generate_with_lora(
    "계약서의 제3조를 분석해주세요",
    adapter_name="legal-assistant"
)
```

### 5. 모바일에서 Fine-tuned 모델 사용

```swift
// iOS - Fine-tuned 모델 API 호출
import Foundation

struct FineTunedModelService {
    private let modelId: String
    private let apiClient: APIClient

    // 엔드포인트 상수
    private static let chatEndpoint = "/v1/chat/completions"

    init(modelId: String, apiClient: APIClient) {
        self.modelId = modelId  // "ft:gpt-4o-mini:company:support:abc123"
        self.apiClient = apiClient
    }

    func chat(messages: [ChatMessage], temperature: Double = 0.7) async throws -> String {
        let request = ChatRequest(
            model: modelId,
            messages: messages,
            temperature: temperature,
            maxTokens: 500
        )

        let response: ChatResponse = try await apiClient.post(
            Self.chatEndpoint,
            body: request
        )

        return response.choices[0].message.content
    }

    // 스트리밍 응답
    func chatStream(messages: [ChatMessage]) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            Task {
                let request = ChatRequest(
                    model: modelId,
                    messages: messages,
                    stream: true
                )

                for try await chunk in apiClient.streamPost(Self.chatEndpoint, body: request) {
                    if let content = chunk.choices.first?.delta.content {
                        continuation.yield(content)
                    }
                }
                continuation.finish()
            }
        }
    }
}

// 사용
let supportBot = FineTunedModelService(
    modelId: "ft:gpt-4o-mini:company:support:abc123",
    apiClient: apiClient
)

let response = try await supportBot.chat(messages: [
    ChatMessage(role: "user", content: "결제가 안 돼요")
])
```

---

## 5. 장단점

### 장점

| 항목 | 설명 |
|------|------|
| **도메인 특화** | 특정 분야에서 월등한 성능 |
| **일관된 스타일** | 원하는 톤앤매너 유지 |
| **응답 속도** | RAG 대비 빠름 (검색 단계 없음) |
| **비용 절감** | 작은 모델로도 특화 작업 수행 가능 |
| **프롬프트 최소화** | 긴 시스템 프롬프트 불필요 |
| **지식 내재화** | 모델 자체에 지식이 포함됨 |

### 단점

| 항목 | 설명 |
|------|------|
| **데이터 필요** | 고품질 학습 데이터 수백~수천 개 필요 |
| **정적 지식** | 학습 후 지식 업데이트 어려움 |
| **비용/시간** | 학습에 GPU와 시간 필요 |
| **과적합 위험** | 다양성 감소, 특정 패턴만 학습 |
| **평가 어려움** | 성능 측정 기준이 모호할 수 있음 |
| **모델 관리** | 여러 버전 관리 복잡도 |

### RAG vs Fine-tuning 의사결정 트리

```
                시작
                 │
                 ▼
        ┌───────────────────┐
        │ 데이터가 자주     │
        │ 업데이트 되는가?  │
        └─────────┬─────────┘
                  │
          ┌───────┴───────┐
          │               │
         Yes              No
          │               │
          ▼               ▼
       [RAG]     ┌───────────────────┐
                 │ 특정 스타일/형식  │
                 │ 이 필요한가?      │
                 └─────────┬─────────┘
                           │
                   ┌───────┴───────┐
                   │               │
                  Yes              No
                   │               │
                   ▼               ▼
            [Fine-tuning]   ┌───────────────────┐
                            │ 복잡한 추론이     │
                            │ 필요한가?         │
                            └─────────┬─────────┘
                                      │
                              ┌───────┴───────┐
                              │               │
                             Yes              No
                              │               │
                              ▼               ▼
                       [Fine-tuning]   [Prompt Engineering]
                       또는 둘 다 조합

실무 권장 접근:
┌─────────────────────────────────────────────────────────────────┐
│  1단계: Prompt Engineering으로 시작                            │
│  2단계: 부족하면 RAG 추가                                       │
│  3단계: 그래도 부족하면 Fine-tuning 검토                        │
│  최종: RAG + Fine-tuned 모델 조합이 가장 강력                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 내 생각

```
이 섹션은 학습 후 직접 작성해보세요.

- Fine-tuning이 해결하는 핵심 문제는?
- 우리 서비스에서 Fine-tuning이 필요한 부분은?
- 학습 데이터를 어떻게 확보할 수 있을까?
- RAG와 Fine-tuning 중 무엇을 먼저 시도할까?
```

---

## 7. 추가 질문

### 초급

1. Fine-tuning과 Pre-training의 차이는 무엇인가요?

> **답변**: Pre-training은 모델을 처음부터 대규모 데이터(수조 토큰)로 훈련하여 언어의 기본 구조와 일반 지식을 학습시키는 과정입니다. 수개월의 시간과 수백~수천만 달러의 비용이 들며, GPT-4나 Llama 같은 기초 모델을 만드는 단계입니다. 반면 Fine-tuning은 이미 Pre-training된 모델을 특정 작업이나 도메인에 맞게 추가 학습시키는 과정입니다. 수백~수천 개의 예시 데이터로 몇 시간~며칠 내에 완료되며, 비용도 훨씬 적습니다. 비유하면 Pre-training은 "대학교 4년 교육", Fine-tuning은 "회사 입사 후 OJT 교육"과 같습니다. 모바일 개발자는 보통 Fine-tuning만 수행하며, Pre-training된 기초 모델을 API나 오픈소스로 사용합니다.

2. LoRA가 Full Fine-tuning보다 효율적인 이유는?

> **답변**: LoRA(Low-Rank Adaptation)가 효율적인 이유는 학습해야 할 파라미터 수를 획기적으로 줄이기 때문입니다. Full Fine-tuning은 모델의 모든 파라미터(7B 모델이면 70억 개)를 업데이트하여 GPU 메모리를 많이 사용하고(~60GB VRAM), 각 Fine-tuned 버전마다 전체 모델을 저장해야 합니다. LoRA는 원본 가중치를 동결하고, 작은 "어댑터" 행렬만 학습합니다. 예를 들어 7B 모델에서 LoRA는 약 0.1-1%의 파라미터만 학습하여 ~16GB VRAM으로도 가능합니다. 또한 여러 용도의 LoRA 어댑터(고객지원용, 법률용 등)를 수십 MB 크기로 따로 저장하고, 런타임에 동적으로 교체할 수 있습니다. 성능은 Full Fine-tuning의 90-95% 수준으로, 대부분의 실무 사례에서 충분합니다.

3. Fine-tuning에 필요한 최소 데이터 양은?

> **답변**: Fine-tuning에 필요한 데이터 양은 목적에 따라 다릅니다. **OpenAI 권장**: 최소 10개 예시, 권장 50-100개, 이상적으로 수백~수천 개입니다. **실무 경험치**: 간단한 스타일/톤 조정은 50-100개로 효과를 볼 수 있고, 새로운 지식 학습이나 복잡한 추론은 500-1000개 이상이 필요합니다. **데이터 품질이 양보다 중요**: 100개의 고품질 예시가 1000개의 저품질 예시보다 낫습니다. 고품질 기준은 (1) 일관된 형식, (2) 정확한 정보, (3) 다양한 케이스 커버, (4) 원하는 스타일 반영입니다. 모바일 앱의 고객 지원 챗봇이라면, 실제 고객 문의 100-200개를 이상적인 답변과 함께 정리하는 것으로 시작할 수 있습니다.

### 중급

4. 과적합(Overfitting)을 방지하는 방법은?

> **답변**: Fine-tuning 시 과적합을 방지하는 방법입니다. (1) **데이터 다양성 확보**: 다양한 패턴과 케이스를 포함하고, 유사한 예시가 반복되지 않게 합니다. (2) **Epoch 수 조절**: 보통 2-5 epoch이 적절하며, Validation loss가 증가하기 시작하면 Early stopping을 적용합니다. (3) **Learning Rate 낮추기**: 기본값의 50-100%로 시작하고, 필요시 더 낮춥니다. (4) **Dropout 사용**: LoRA에서 lora_dropout=0.05~0.1을 설정합니다. (5) **데이터 증강**: 동일한 의미의 다른 표현으로 데이터를 늘립니다. (6) **검증 세트 분리**: 학습 데이터의 10-20%를 검증용으로 분리하여 모니터링합니다. (7) **정규화**: Weight decay를 적용합니다. 과적합의 징후는 학습 데이터에서는 완벽하지만 새로운 질문에 엉뚱한 답변을 하거나, 학습 예시를 그대로 복사하는 경우입니다.

5. Learning Rate와 Epoch 수는 어떻게 결정하나요?

> **답변**: Learning Rate와 Epoch 설정 가이드입니다. **Learning Rate**: (1) OpenAI Fine-tuning은 자동 조정되지만, multiplier 0.5-2.0으로 조절 가능합니다. (2) 오픈소스 LoRA는 1e-5 ~ 5e-4 범위에서 시작하고, 보통 2e-4가 좋은 시작점입니다. (3) 큰 데이터셋은 높은 LR, 작은 데이터셋은 낮은 LR이 안전합니다. **Epoch**: (1) 데이터가 적으면(100개 미만) 3-5 epoch, 많으면(1000개 이상) 1-3 epoch이 적절합니다. (2) Validation loss를 모니터링하여 증가 시작 전에 중단합니다. **실험 전략**: 먼저 작은 서브셋(10-20%)으로 다양한 설정을 빠르게 테스트하고, 최적 설정을 찾은 후 전체 데이터로 학습합니다. Weights & Biases 같은 도구로 여러 실험을 비교하면 효율적입니다.

6. Multi-task Fine-tuning이란 무엇인가요?

> **답변**: Multi-task Fine-tuning은 하나의 모델을 여러 작업(task)에 동시에 학습시키는 기법입니다. **예시**: 고객 지원 챗봇 하나가 (1) 일반 문의 응대, (2) 환불 처리 안내, (3) 기술 지원, (4) 감정 분석을 모두 수행하도록 학습합니다. **장점**: (1) 각 작업 간 지식 공유로 전체 성능 향상 (2) 여러 모델 대신 하나만 관리 (3) 일반화 능력 향상으로 과적합 감소. **구현 방법**: 각 작업을 구분하는 프롬프트 접두사를 사용합니다. 예: "[일반문의] 배송 얼마나 걸려요?", "[환불] 환불하고 싶어요". 모델은 접두사를 보고 적절한 응답 스타일을 선택합니다. **주의점**: 작업 간 데이터 양의 균형을 맞춰야 하며, 상충되는 목표가 있는 작업은 분리하는 것이 좋습니다.

### 심화

7. DPO(Direct Preference Optimization)란 무엇인가요?

> **답변**: DPO는 인간 선호도에 맞게 모델을 학습시키는 기법으로, 기존 RLHF(Reinforcement Learning from Human Feedback)의 간소화 버전입니다. **RLHF의 문제**: (1) Reward Model을 별도로 학습해야 함 (2) 강화학습(PPO) 알고리즘이 불안정 (3) 구현이 복잡하고 계산 비용이 높음. **DPO의 해결**: (1) Reward Model 없이 직접 선호도 학습 (2) 단순한 분류 손실 함수 사용 (3) 안정적이고 효율적. **작동 방식**: "선호되는 답변"과 "선호되지 않는 답변" 쌍을 제공하면, 모델이 선호되는 답변의 확률을 높이도록 학습합니다. **모바일 앱 적용**: 챗봇의 답변 품질을 높일 때, 사용자 피드백(좋아요/싫어요)을 수집하여 DPO 데이터로 활용할 수 있습니다. 현재 Claude, GPT-4 등 최신 모델들이 DPO 또는 유사 기법으로 학습됩니다.

8. Instruction Tuning과 RLHF의 차이는?

> **답변**: Instruction Tuning과 RLHF는 모델을 사람의 의도에 맞게 학습시키는 다른 단계입니다. **Instruction Tuning (SFT)**: 지시문-응답 쌍으로 지도 학습합니다. "이메일을 요약해줘" → "요약 결과" 형태의 데이터로 모델이 지시를 따르도록 학습합니다. 장점으로 구현이 간단하고, 기본적인 지시 수행 능력을 부여합니다. 단점으로 "좋은" 응답과 "나쁜" 응답을 구분하지 못합니다. **RLHF**: 인간 선호도 피드백으로 강화학습합니다. 같은 질문에 대한 여러 응답 중 어떤 것이 더 좋은지 평가하여 학습합니다. 장점으로 응답 품질, 안전성, 유용성을 크게 향상시킵니다. 단점으로 구현이 복잡하고 비용이 높습니다. **일반적 학습 순서**: Pre-training → Instruction Tuning → RLHF/DPO. 최근에는 DPO가 RLHF를 대체하는 추세입니다.

9. Catastrophic Forgetting을 방지하는 방법은?

> **답변**: Catastrophic Forgetting은 Fine-tuning 시 새로운 작업을 학습하면서 기존에 알던 지식을 잊어버리는 현상입니다. **방지 방법**: (1) **낮은 Learning Rate**: 기존 지식을 크게 바꾸지 않도록 작은 LR(1e-5 ~ 5e-5) 사용. (2) **LoRA/어댑터 방식**: 원본 가중치를 동결하므로 근본적으로 방지됨. (3) **Regularization**: 가중치가 원본에서 크게 벗어나지 않도록 L2 정규화 적용. (4) **데이터 리허설**: 새 데이터와 함께 기존 작업 데이터도 일부 포함하여 학습. (5) **Elastic Weight Consolidation (EWC)**: 중요한 가중치에는 큰 변화를 주지 않도록 제약. (6) **점진적 학습**: 한 번에 큰 변화보다 작은 단계로 나누어 학습. **실무 팁**: Fine-tuning 후 반드시 기존 기능(일반 대화, 상식 질문 등)이 유지되는지 테스트하세요. LoRA 사용이 가장 효과적인 방지책입니다.

### 실습 과제

```
과제 1: OpenAI Fine-tuning
- 100개 대화 데이터로 고객 지원 봇 학습
- 기본 모델과 성능 비교

과제 2: LoRA Fine-tuning
- Hugging Face로 Llama 모델 Fine-tuning
- 다양한 rank 값 실험 (4, 8, 16, 32)

과제 3: 평가 파이프라인
- Fine-tuned 모델 평가 지표 설계
- A/B 테스트 구현
```

### 실습 과제 예시 코드

```python
# 과제 3: Fine-tuning 평가 파이프라인
from dataclasses import dataclass
import json
import random

@dataclass
class EvaluationResult:
    """평가 결과 데이터 클래스"""
    model_id: str
    accuracy: float
    relevance_score: float
    style_consistency: float
    latency_ms: float
    cost_per_request: float

class FineTuningEvaluator:
    def __init__(self, test_dataset: list[dict]):
        self.test_data = test_dataset
        self.results = {}

    def evaluate_model(self, model_id: str, inference_fn) -> EvaluationResult:
        """모델 성능 평가"""
        correct = 0
        total_relevance = 0
        total_latency = 0

        for example in self.test_data:
            import time
            start = time.time()

            response = inference_fn(example["input"])

            latency = (time.time() - start) * 1000  # ms

            # 정확도 평가 (예상 답변 포함 여부)
            if any(keyword in response for keyword in example.get("expected_keywords", [])):
                correct += 1

            # 관련성 점수 (LLM-as-judge 방식)
            relevance = self._score_relevance(example["input"], response)
            total_relevance += relevance

            total_latency += latency

        n = len(self.test_data)
        result = EvaluationResult(
            model_id=model_id,
            accuracy=correct / n,
            relevance_score=total_relevance / n,
            style_consistency=self._evaluate_style_consistency(model_id, inference_fn),
            latency_ms=total_latency / n,
            cost_per_request=self._estimate_cost(model_id)
        )

        self.results[model_id] = result
        return result

    def _score_relevance(self, question: str, answer: str) -> float:
        """LLM으로 관련성 점수 평가 (1-5점)"""
        # 실제로는 GPT-4 등을 judge로 사용
        # 여기서는 간단히 길이 기반 휴리스틱
        if len(answer) < 50:
            return 2.0
        elif len(answer) > 500:
            return 3.5
        else:
            return 4.0

    def _evaluate_style_consistency(self, model_id: str, inference_fn) -> float:
        """스타일 일관성 평가"""
        # 동일 질문에 대한 여러 응답의 일관성 체크
        return 0.85  # 예시 값

    def _estimate_cost(self, model_id: str) -> float:
        """요청당 예상 비용"""
        costs = {
            "gpt-4o-mini": 0.0002,
            "ft:gpt-4o-mini:org:support:abc123": 0.0003,
            "llama-3.1-8b": 0.0001,
        }
        return costs.get(model_id, 0.0005)

    def compare_models(self) -> dict:
        """모델 비교 결과"""
        return {
            model_id: {
                "accuracy": result.accuracy,
                "relevance": result.relevance_score,
                "latency_ms": result.latency_ms,
                "cost": result.cost_per_request
            }
            for model_id, result in self.results.items()
        }

# A/B 테스트 구현
class ABTestManager:
    def __init__(self, models: dict[str, str], traffic_split: dict[str, float]):
        """
        models: {"control": "gpt-4o-mini", "treatment": "ft:gpt-4o-mini:..."}
        traffic_split: {"control": 0.5, "treatment": 0.5}
        """
        self.models = models
        self.traffic_split = traffic_split
        self.results = {name: {"count": 0, "feedback_positive": 0} for name in models}

    def assign_group(self, user_id: str) -> str:
        """사용자를 그룹에 할당 (일관된 할당)"""
        # 사용자 ID 기반 결정론적 할당
        hash_val = hash(user_id) % 100 / 100
        cumulative = 0
        for group, ratio in self.traffic_split.items():
            cumulative += ratio
            if hash_val < cumulative:
                return group
        return list(self.models.keys())[0]

    def get_model_for_user(self, user_id: str) -> str:
        """사용자에게 할당된 모델 반환"""
        group = self.assign_group(user_id)
        return self.models[group]

    def record_feedback(self, user_id: str, positive: bool):
        """피드백 기록"""
        group = self.assign_group(user_id)
        self.results[group]["count"] += 1
        if positive:
            self.results[group]["feedback_positive"] += 1

    def get_statistics(self) -> dict:
        """A/B 테스트 통계"""
        stats = {}
        for group, data in self.results.items():
            if data["count"] > 0:
                stats[group] = {
                    "total_requests": data["count"],
                    "positive_rate": data["feedback_positive"] / data["count"],
                    "model": self.models[group]
                }
        return stats

# 사용 예시
ab_test = ABTestManager(
    models={
        "control": "gpt-4o-mini",
        "treatment": "ft:gpt-4o-mini:org:support:abc123"
    },
    traffic_split={"control": 0.5, "treatment": 0.5}
)

# 사용자 요청 처리
user_id = "user_12345"
model_to_use = ab_test.get_model_for_user(user_id)
print(f"User {user_id} assigned to model: {model_to_use}")
```

---

## 8. 실무 운영 시 주의사항

### Fine-tuning 체크리스트

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Fine-tuning 체크리스트                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 데이터 준비                                                         │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  □ 최소 100개 이상의 고품질 예시 확보                         │   │
│     │  □ 데이터 형식 검증 (JSONL, 메시지 구조)                      │   │
│     │  □ 중복 및 모순되는 데이터 제거                               │   │
│     │  □ 민감 정보(PII) 마스킹 또는 제거                            │   │
│     │  □ Train/Validation 분리 (80/20 또는 90/10)                   │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. 학습 전                                                             │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  □ 베이스 모델 선택 (비용, 성능, 라이선스 고려)               │   │
│     │  □ 하이퍼파라미터 초기값 설정                                 │   │
│     │  □ 예상 비용 및 시간 계산                                     │   │
│     │  □ 실험 추적 도구 설정 (W&B, MLflow)                          │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. 학습 중 모니터링                                                    │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  □ Training/Validation loss 추이                              │   │
│     │  □ 과적합 징후 (val loss 상승)                                │   │
│     │  □ 중간 체크포인트 저장                                       │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  4. 학습 후 평가                                                        │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  □ 테스트 세트로 정량 평가                                    │   │
│     │  □ 베이스 모델 대비 성능 비교                                 │   │
│     │  □ 엣지 케이스 및 실패 사례 분석                              │   │
│     │  □ 기존 기능 유지 확인 (Catastrophic Forgetting 체크)        │   │
│     │  □ Human evaluation 수행                                      │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  5. 배포                                                                │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  □ A/B 테스트 설정                                            │   │
│     │  □ 롤백 계획 수립                                             │   │
│     │  □ 모니터링 대시보드 구성                                     │   │
│     │  □ 비용 알림 설정                                             │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Fine-tuning 비용 계산

```python
# Fine-tuning 비용 계산기
class FineTuningCostCalculator:
    # OpenAI Fine-tuning 가격 (2024-2025)
    OPENAI_PRICING = {
        "gpt-4o-mini-2024-07-18": {
            "training": 3.00,  # $/1M tokens
            "inference_input": 0.30,
            "inference_output": 1.20
        },
        "gpt-4o-2024-08-06": {
            "training": 25.00,
            "inference_input": 3.75,
            "inference_output": 15.00
        }
    }

    def estimate_training_cost(
        self,
        model: str,
        num_examples: int,
        avg_tokens_per_example: int,
        epochs: int = 3
    ) -> dict:
        """학습 비용 추정"""
        pricing = self.OPENAI_PRICING.get(model)
        if not pricing:
            return {"error": f"Unknown model: {model}"}

        total_tokens = num_examples * avg_tokens_per_example * epochs
        training_cost = (total_tokens / 1_000_000) * pricing["training"]

        return {
            "model": model,
            "num_examples": num_examples,
            "total_tokens": total_tokens,
            "epochs": epochs,
            "training_cost_usd": round(training_cost, 2),
            "estimated_time_minutes": max(10, total_tokens // 100_000)  # 대략적 추정
        }

    def estimate_inference_cost(
        self,
        model: str,
        requests_per_day: int,
        avg_input_tokens: int,
        avg_output_tokens: int
    ) -> dict:
        """월간 추론 비용 추정"""
        pricing = self.OPENAI_PRICING.get(model)
        if not pricing:
            return {"error": f"Unknown model: {model}"}

        daily_input_tokens = requests_per_day * avg_input_tokens
        daily_output_tokens = requests_per_day * avg_output_tokens

        daily_cost = (
            (daily_input_tokens / 1_000_000) * pricing["inference_input"] +
            (daily_output_tokens / 1_000_000) * pricing["inference_output"]
        )

        return {
            "model": model,
            "requests_per_day": requests_per_day,
            "daily_cost_usd": round(daily_cost, 2),
            "monthly_cost_usd": round(daily_cost * 30, 2),
            "yearly_cost_usd": round(daily_cost * 365, 2)
        }

# 사용 예시
calculator = FineTuningCostCalculator()

# 학습 비용
training_cost = calculator.estimate_training_cost(
    model="gpt-4o-mini-2024-07-18",
    num_examples=500,
    avg_tokens_per_example=500,
    epochs=3
)
print(f"예상 학습 비용: ${training_cost['training_cost_usd']}")

# 추론 비용
inference_cost = calculator.estimate_inference_cost(
    model="gpt-4o-mini-2024-07-18",
    requests_per_day=10000,
    avg_input_tokens=200,
    avg_output_tokens=300
)
print(f"예상 월간 비용: ${inference_cost['monthly_cost_usd']}")
```
