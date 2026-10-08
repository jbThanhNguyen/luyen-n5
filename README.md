# Vũ Trụ Từ Vựng N5 · N4 🪐

Web app ôn từ vựng, kanji, ngữ pháp JLPT N5 và N4 theo kiểu Quizlet, giao diện tối chủ đề vũ trụ. Không cần cài đặt, chạy được offline.

🔗 **Dùng thử:** https://luyenn5.netlify.app

## Tính năng

- **Chọn cấp N5 / N4 / N5+N4** ở góc trên. Lựa chọn được nhớ lại; cũng có thể mở thẳng bằng link `?level=n4`. Chế độ N5+N4 trộn cả hai bộ để ôn tổng.
- **Trắc nghiệm 293 từ N5 và 798 từ N4**: 4 đáp án, chọn hướng Nhật → Việt, Việt → Nhật hoặc trộn cả hai. Lọc từ theo phần, chữ cái đầu, từ loại hoặc nhóm "từ hay sai".
- **Hỏi lại câu sai**: từ trả lời sai sẽ được hỏi lại sau vài câu, đến khi đúng thì thôi.
- **Đáp án có ví dụ**: mỗi từ N5 kèm một câu ví dụ, có hiragana đặt kế bên kanji, romaji và nghĩa tiếng Việt.
- **Bảng chữ**: Hiragana và Katakana (âm cơ bản, âm đục, âm ghép), có chế độ che romaji để tự kiểm tra.
- **246 Kanji**: 79 chữ N5 chuẩn và 167 chữ N4. Ở chế độ N5+N4 có nhãn cấp và lọc được theo cấp. Mỗi chữ có âm Hán Việt, âm On, âm Kun, ví dụ, và các từ vựng chứa chữ đó.
- **Ngữ pháp: 37 mẫu N5 (75 câu luyện) và 85 mẫu N4 (137 câu luyện)**. N4 theo Minna no Nihongo bài 26–50, lọc được theo từng bài. Mỗi mẫu có ví dụ (N4 kèm furigana), romaji, ghi chú; trắc nghiệm điền chỗ trống, sai thì hỏi lại, kèm giải thích mẫu ngữ pháp.
- **Dữ liệu cho học máy**: mỗi lần trả lời được ghi thành một dòng (30 cột, có cột `level`). Bấm nút để xuất file CSV. Khung này chỉ hiện với chủ app; người khác dùng app bình thường nhưng không thấy và không xuất được dữ liệu.

## Cách dùng

1. Vào link dùng thử ở trên, hoặc tải `index.html` **cùng thư mục `data/`** về rồi mở `index.html` bằng trình duyệt (Chrome, Edge, Firefox…).
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
| Định danh | `attempt_id`, `session_id`, `learner`, `timestamp`, `level` |
| Từ vựng | `word_key`, `romaji`, `kana`, `kanji`, `meaning`, `pos`, `has_kanji`, `kana_len` |
| Ngữ cảnh câu hỏi | `direction`, `romaji_shown`, `q_index`, `repeat_in_session`, `hour`, `weekday` |
| Lịch sử trước đó | `prior_attempts`, `prior_correct`, `prior_wrong`, `prior_last_correct`, `sec_since_last` |
| Kết quả ⚠️ | `correct`, `chosen_pos`, `correct_pos`, `chosen_text`, `dont_know`, `response_ms` |

`level` là cấp của từ (`n5` / `n4`); các dòng ghi trước khi có cột này được điền `n5`. `word_key` của từ N4 có tiền tố `n4:` để không lẫn với thống kê N5.

⚠️ Các cột ở nhóm **Kết quả** chỉ có sau khi đã trả lời, và `chosen_pos` so với `correct_pos` là ra luôn đáp án. Vì vậy đừng đưa nhóm này vào feature khi train model đoán *trước* lúc hỏi. Nếu gộp dữ liệu của nhiều người, nên chia train/test theo `learner` bằng `GroupShuffleSplit`.

## Cấu trúc thư mục

```
index.html     giao diện + logic (không chứa dữ liệu)
data/n5.js     window.JLPT.n5 = { words, kanji, grammar }
data/n4.js     window.JLPT.n4 = { words, kanji, grammar }
```

Muốn sửa hay thêm từ, kanji, ngữ pháp thì chỉ cần sửa file trong `data/`, mỗi mục một dòng. Định dạng từng mảng ghi ở đầu mỗi file. Dữ liệu dùng file `.js` (không phải `.json`) để vẫn mở được bằng `file://` khi chạy offline.

## Nguồn dữ liệu

- `Tu_vung_JLPT_N5_A-Z.xlsx`: 295 từ, còn 293 từ sau khi gộp các dòng trùng. Đã sửa romaji của 九月 thành `kugatsu`.
- `Kanji_N5_co_ban.xlsx`: 49 kanji ban đầu, bổ sung thêm 39 chữ cho đủ bộ 79 chữ N5 chuẩn (tổng 88), thêm cột Hán Việt và Cấp độ.
- `N5_Grammar_Tong_Hop.xlsx`: 46 mẫu gốc cộng 18 mẫu bổ sung, gộp lại còn 37 mẫu (cột `Merged_From` ghi mẫu cũ nằm ở đâu).
- N4: `JLPT_N4_nouns/verbs/adjectives/adverbs_A-Z.xlsx` (759 mục) đã sửa lỗi (romaji 歩く, ô có 2 dạng từ, mẫu ngữ pháp lẫn trong file trạng từ, dòng trùng) và bổ sung 50 từ còn thiếu (liên từ, trạng từ, động từ, danh từ する…), còn 798 từ. Từ N4 chưa có câu ví dụ.
- `JLPT_N4_kanji_day_du.xlsx`: 167 kanji N4, có Hán Việt, âm On/Kun và từ ví dụ; âm đọc đối chiếu KANJIDIC2, từ ví dụ đối chiếu JMdict (EDRDG, CC BY-SA 4.0).
- Câu ví dụ được viết riêng cho bộ từ này. Cách đọc và romaji đã được kiểm tra bằng máy.
