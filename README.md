# Vũ Trụ Từ Vựng N5 🪐

Web app ôn từ vựng JLPT N5 theo kiểu Quizlet, giao diện tối chủ đề vũ trụ. Chỉ gồm một file HTML, chạy offline không cần cài đặt.

🔗 **Dùng thử:** https://luyenn5.netlify.app

## Tính năng

- **Trắc nghiệm 293 từ N5**: 4 đáp án, chọn hướng Nhật → Việt, Việt → Nhật hoặc trộn cả hai. Lọc từ theo phần, chữ cái đầu, từ loại hoặc nhóm "từ hay sai".
- **Hỏi lại câu sai**: từ trả lời sai sẽ được hỏi lại sau vài câu, đến khi đúng thì thôi.
- **Đáp án có ví dụ**: mỗi đáp án kèm một câu ví dụ, có hiragana đặt kế bên kanji, romaji và nghĩa tiếng Việt.
- **Bảng chữ**: Hiragana và Katakana (âm cơ bản, âm đục, âm ghép), có chế độ che romaji để tự kiểm tra.
- **88 Kanji**: đủ 79 chữ N5 chuẩn, cộng 9 chữ N4 hay gặp (có nhãn N4, lọc được theo cấp độ). Mỗi chữ có âm Hán Việt, âm On, âm Kun, ví dụ, và các từ vựng chứa chữ đó.
- **37 mẫu ngữ pháp N5** chia 6 nhóm: mỗi mẫu có ví dụ, romaji, ghi chú. Có 75 câu trắc nghiệm điền chỗ trống, sai thì hỏi lại, kèm giải thích mẫu ngữ pháp.
- **Dữ liệu cho học máy**: mỗi lần trả lời được ghi thành một dòng (29 cột). Bấm nút để xuất file CSV. Khung này chỉ hiện với chủ app; người khác dùng app bình thường nhưng không thấy và không xuất được dữ liệu.

## Cách dùng

1. Vào link dùng thử ở trên, hoặc tải `index.html` về rồi mở bằng trình duyệt (Chrome, Edge, Firefox…).
2. Chọn hướng câu hỏi, phạm vi từ và số từ mỗi lượt, rồi bấm **Bắt đầu**.
3. Phím tắt: `1`–`4` chọn đáp án · `0` không biết · `Enter` qua câu tiếp.

> Tiến độ học và log chỉ lưu trong trình duyệt đang dùng. Xoá dữ liệu trình duyệt hoặc đổi máy thì sẽ mất, nên nhớ tải CSV về định kỳ.

## Dữ liệu học máy

Khung **Dữ liệu cho học máy** chỉ hiện với chủ app. Nhập **Tên người học** trước khi làm bài, sau đó bấm **Tải CSV** ở màn hình chính.

```python
import pandas as pd, glob
df = pd.concat([pd.read_csv(f) for f in glob.glob("n5_quiz_log_*.csv")], ignore_index=True)
df = df.drop_duplicates("attempt_id")
y = df["correct"]   # nhãn: 1 = đúng, 0 = sai
```

| Nhóm cột | Cột |
|---|---|
| Định danh | `attempt_id`, `session_id`, `learner`, `timestamp` |
| Từ vựng | `word_key`, `romaji`, `kana`, `kanji`, `meaning`, `pos`, `has_kanji`, `kana_len` |
| Ngữ cảnh câu hỏi | `direction`, `romaji_shown`, `q_index`, `repeat_in_session`, `hour`, `weekday` |
| Lịch sử trước đó | `prior_attempts`, `prior_correct`, `prior_wrong`, `prior_last_correct`, `sec_since_last` |
| Kết quả ⚠️ | `correct`, `chosen_pos`, `correct_pos`, `chosen_text`, `dont_know`, `response_ms` |

⚠️ Các cột ở nhóm **Kết quả** chỉ có sau khi đã trả lời, và `chosen_pos` so với `correct_pos` là ra luôn đáp án. Vì vậy đừng đưa nhóm này vào feature khi train model đoán *trước* lúc hỏi. Nếu gộp dữ liệu của nhiều người, nên chia train/test theo `learner` bằng `GroupShuffleSplit`.

## Nguồn dữ liệu

- `Tu_vung_JLPT_N5_A-Z.xlsx`: 295 từ, còn 293 từ sau khi gộp các dòng trùng. Đã sửa romaji của 九月 thành `kugatsu`.
- `Kanji_N5_co_ban.xlsx`: 49 kanji ban đầu, bổ sung thêm 39 chữ cho đủ bộ 79 chữ N5 chuẩn (tổng 88), thêm cột Hán Việt và Cấp độ.
- `N5_Grammar_Tong_Hop.xlsx`: 46 mẫu gốc cộng 18 mẫu bổ sung, gộp lại còn 37 mẫu (cột `Merged_From` ghi mẫu cũ nằm ở đâu).
- Câu ví dụ được viết riêng cho bộ từ này. Cách đọc và romaji đã được kiểm tra bằng máy.
