# Các lệnh bạn tự chạy trên máy (phần cần định danh cá nhân)

## Tiết 1
```
cd yummyapp-cicd
pip install -r requirements.txt
coverage run -m unittest discover -s . -p "test_*.py" > logs/local_test.log 2>&1
coverage report -m >> logs/local_test.log
git init
git branch -M main
git config user.name "MSSV - Ho Va Ten"
git config user.email "mssv@dlu.edu.vn"
git add .
git commit -m "Initial commit: YummyApp"
git remote add origin https://github.com/<ten-ban>/yummyapp-cicd.git
git push -u origin main
```

## Tiết 3 – xem CI đỏ rồi xanh (làm đúng thứ tự)
1. Ban đầu: để test_yummy_app.py chỉ có TestEasyLevel (xóa tạm 2 lớp Medium/Hard), push -> CI đỏ ở bước 5 (~35%).
2. Dán lại 2 lớp TestMediumLevel và TestHardLevel, chạy local, push -> CI xanh.

## Tiết 4
- Break the Build: thêm vào yummy_app.py một dòng sai cú pháp, ví dụ `def hong(:`, push -> bước 4 đỏ; xóa đi, push lại -> xanh.
- Log lịch sử:
```
git log --graph --pretty=format:"%h - %an (%ad): %s" > logs/git_history.log
```
- Log 3: GitHub Actions -> lần chạy xanh cuối -> ba chấm -> Download log archive -> lưu vào logs/github_actions_run.log
- Chụp màn hình tích xanh/đỏ, tải artifact YummyApp-Coverage-Report để đưa vào báo cáo.
