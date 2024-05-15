# ChessAI

## Cách chạy
1. Cài đặt các thư viện cần thiết `python -m pip install -r requirements.txt`.
2. Chạy file main.py để chơi
3. 
## Data 
1. Tôi sử dụng bộ dữ liệu từ Lichess, nơi chứa các trò chơi ở định dạng PGN tiêu chuẩn :<a href="https://database.lichess.org">
2. Quá trình xử lý dữ liệu
   - Tải xuống tệp dữ liệu PGN thô từ <a href="https://database.lichess.org">Lichess</a>.
   - Tải xuống mô-đun `pgn-extract` <a href="https://www.cs.kent.ac.uk/people/staff/djb/pgn-extract/">tại đây</a>.
   - Chạy `pgn-extract --quiet -D --fencomments --fixresulttags -w 100000 -o out.pgn master_db.pgn`. Điều này thêm một FEN sau mỗi lần di chuyển và gói gọn mỗi trò chơi trong một dòng duy nhất.
   - Chạy `extract_fen.py` để trích xuất các nhận xét FEN.
   - `get_moves.py` xác định những ô cờ mà quân cờ di chuyển từ đó và và những ô cờ mà quân cờ di chuyển tới cho mỗi lần di chuyển.
  Xem chi tiết tại thư mục `data_cleaning`

## GUI
GUI được tạo thủ công bằng cách sử dụng `pygame` và` python-chess`.

## Model
AI sử dụng hai mô hình. Cả hai đều nhận được một vị trí trên bảng làm đầu vào và đầu ra là một ma trận `8x8` có xác suất softmax. "from model" dự đoán ô cờ mà quân cờ sẽ bắt đầu di chuyển và "to model" dự đoán ô cờ mà quân cờ sẽ di chuyển đến.

Ví dụ: xem xét vị trí bàn xuất phát và nước đi: `Nf3`. Việc đánh giá bước di chuyển này là tích của giá trị tại bình phương `g1` của "from model" và giá trị tại bình phương `f3` của "to model".
Trong số tất cả các nước đi hợp lệ, tích lớn nhất là nước đi được lựa chọn.

Mạng lưới thần kinh bao gồm sáu lớp tích chập, tiếp theo là hai lớp affine và một lớp đầu ra. Một bản phác thảo chi tiết hơn về kiến trúc có thể được tìm thấy dưới đây:
Model: "model"
'''
__________________________________________________________________________________________________
 Layer (type)                   Output Shape         Param #     Connected to                     
==================================================================================================
 input_1 (InputLayer)           [(None, 8, 8, 12)]   0           []                               
                                                                                                  
 conv2d (Conv2D)                (None, 8, 8, 32)     3488        ['input_1[0][0]']                
                                                                                                  
 batch_normalization (BatchNorm  (None, 8, 8, 32)    128         ['conv2d[0][0]']                 
 alization)                                                                                       
                                                                                                  
 activation (Activation)        (None, 8, 8, 32)     0           ['batch_normalization[0][0]']    
                                                                                                  
 conv2d_1 (Conv2D)              (None, 8, 8, 64)     18496       ['activation[0][0]']             
                                                                                                  
 batch_normalization_1 (BatchNo  (None, 8, 8, 64)    256         ['conv2d_1[0][0]']               
 rmalization)                                                                                     
                                                                                                  
 activation_1 (Activation)      (None, 8, 8, 64)     0           ['batch_normalization_1[0][0]']  
                                                                                                  
 conv2d_2 (Conv2D)              (None, 8, 8, 256)    147712      ['activation_1[0][0]']           
                                                                                                  
 batch_normalization_2 (BatchNo  (None, 8, 8, 256)   1024        ['conv2d_2[0][0]']               
 rmalization)                                                                                     
                                                                                                  
 activation_2 (Activation)      (None, 8, 8, 256)    0           ['batch_normalization_2[0][0]']  
                                                                                                  
 concatenate (Concatenate)      (None, 8, 8, 512)    0           ['activation_2[0][0]',           
                                                                  'activation_2[0][0]']           
                                                                                                  
 conv2d_3 (Conv2D)              (None, 8, 8, 256)    1179904     ['concatenate[0][0]']            
                                                                                                  
 batch_normalization_3 (BatchNo  (None, 8, 8, 256)   1024        ['conv2d_3[0][0]']               
 rmalization)                                                                                     
                                                                                                  
 activation_3 (Activation)      (None, 8, 8, 256)    0           ['batch_normalization_3[0][0]']  
                                                                                                  
 concatenate_1 (Concatenate)    (None, 8, 8, 320)    0           ['activation_3[0][0]',           
                                                                  'activation_1[0][0]']           
                                                                                                  
 conv2d_4 (Conv2D)              (None, 8, 8, 256)    737536      ['concatenate_1[0][0]']          
                                                                                                  
 batch_normalization_4 (BatchNo  (None, 8, 8, 256)   1024        ['conv2d_4[0][0]']               
 rmalization)                                                                                     
                                                                                                  
 activation_4 (Activation)      (None, 8, 8, 256)    0           ['batch_normalization_4[0][0]']  
                                                                                                  
 concatenate_2 (Concatenate)    (None, 8, 8, 288)    0           ['activation_4[0][0]',           
                                                                  'activation[0][0]']             
                                                                                                  
 conv2d_5 (Conv2D)              (None, 8, 8, 256)    663808      ['concatenate_2[0][0]']          
                                                                                                  
 batch_normalization_5 (BatchNo  (None, 8, 8, 256)   1024        ['conv2d_5[0][0]']               
 rmalization)                                                                                     
                                                                                                  
 activation_5 (Activation)      (None, 8, 8, 256)    0           ['batch_normalization_5[0][0]']  
                                                                                                  
 dense (Dense)                  (None, 8, 8, 256)    65792       ['activation_5[0][0]']           
                                                                                                  
 batch_normalization_6 (BatchNo  (None, 8, 8, 256)   1024        ['dense[0][0]']                  
 rmalization)                                                                                     
                                                                                                  
 dense_1 (Dense)                (None, 8, 8, 64)     16448       ['batch_normalization_6[0][0]']  
                                                                                                  
 batch_normalization_7 (BatchNo  (None, 8, 8, 64)    256         ['dense_1[0][0]']                
 rmalization)                                                                                     
                                                                                                  
 dense_2 (Dense)                (None, 8, 8, 1)      65          ['batch_normalization_7[0][0]']  
                                                                                                  
 batch_normalization_8 (BatchNo  (None, 8, 8, 1)     4           ['dense_2[0][0]']                
 rmalization)                                                                                     
                                                                                                  
 softmax (Softmax)              (None, 8, 8, 1)      0           ['batch_normalization_8[0][0]']  
                                                                                                  
==================================================================================================
Total params: 2,839,013
Trainable params: 2,836,131
Non-trainable params: 2,882
__________________________________________________________________________________________________
```
