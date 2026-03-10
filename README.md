# SimpleNote — Cosmos SDK Boilerplate

> Blockchain module đơn giản nhất: lưu note on-chain với timestamp bất biến.

---

## Cấu trúc project

```
simplenote/
├── x/note/                  ← module chính
│   ├── types/
│   │   ├── types.go         ← Note struct, MsgSubmitNote
│   │   ├── errors.go        ← custom errors
│   │   └── keys.go          ← KV store key helpers
│   ├── keeper/
│   │   ├── keeper.go        ← business logic (ĐỌC FILE NÀY TRƯỚC)
│   │   └── keeper_test.go   ← unit tests
│   ├── client/cli/
│   │   └── tx.go            ← CLI commands
│   └── module.go            ← overview + luồng xử lý
└── go.mod
```

---

## Luồng dữ liệu (đọc theo thứ tự)

```
1. types/types.go   → hiểu Note là gì, MsgSubmitNote trông như thế nào
2. types/keys.go    → hiểu cách lưu vào KV store
3. keeper/keeper.go → hiểu business logic xử lý msg
4. client/cli/tx.go → hiểu user tương tác như thế nào
```

---

## Chạy tests ngay (không cần chain thật)

```bash
cd x/note/keeper
go test ./... -v
```

Output mong đợi:
```
--- PASS: TestSubmitNote_Success
--- PASS: TestSubmitNote_AutoIncrement
--- PASS: TestGetNote_NotFound
--- PASS: TestSubmitNote_EmptyContent
--- PASS: TestSubmitNote_ContentTooLong
--- PASS: TestNoteCount
```

---

## Bước tiếp theo (TODO trong code)

### Bước 1 — Chạy tests, đọc hiểu keeper.go ✅
Đây là phần bạn có ngay bây giờ.

### Bước 2 — Kết nối với Cosmos SDK thật
```bash
# Cài ignite CLI (scaffolding tool của Cosmos)
curl https://get.ignite.com/cli! | bash

# Tạo chain mới từ template
ignite scaffold chain simplenote

# Scaffold module note
ignite scaffold module note

# Scaffold message
ignite scaffold message submitNote content:string
```

### Bước 3 — Thêm gRPC Query Server
```go
// Tạo query.go trong keeper/
func (k Keeper) GetNote(ctx context.Context, req *types.QueryGetNoteRequest) (*types.QueryGetNoteResponse, error) {
    note, err := k.GetNoteFromStore(req.Id)
    ...
}
```

### Bước 4 — Chạy local testnet
```bash
ignite chain serve
# → chain chạy tại localhost:26657
# → REST API tại localhost:1317
```

### Bước 5 — Submit note qua CLI
```bash
simplenoted tx note submit-note "Hello blockchain" --from mykey
simplenoted query note get-note 1
```

---

## Concepts cần hiểu

| Concept | File liên quan | Giải thích ngắn |
|---|---|---|
| **Module** | `module.go` | Unit độc lập, giống package trong Go |
| **Message** | `types/types.go` | Request gửi lên chain, cần ký bằng private key |
| **Keeper** | `keeper/keeper.go` | Controller, nơi duy nhất được ghi state |
| **KV Store** | `types/keys.go` | Database của chain, key-value |
| **Handler** | `keeper.SubmitNote()` | Xử lý message → cập nhật state |
| **Query** | `cli/tx.go` | Đọc state, không cần ký, miễn phí |

---

## Sau khi thành thạo Simple Note → học Voting System

Voting System thêm 2 concepts mới:
- **Deadline** — logic dựa vào block height
- **One vote per address** — validation phức tạp hơn
- **Aggregate state** — đếm yes/no votes
