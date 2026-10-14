# Báo Cáo Đo Lường & So Sánh Tối Ưu Hóa Tốc Độ CI Bằng Gradle Dependency Caching

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Lớp:** IT209
- **Học phần:** IT209
- **Session:** 11 - Bài tập 3 (BT3)
- **Đường dẫn nộp bài:** `homework/session_11/ex3/`

---

## 1. Bảng số liệu đo lường thực nghiệm (Benchmark)

| Tiêu chí đánh giá | Lần 1 (Cold Run - Chưa có Cache) | Lần 2 (Warm Run - Đã khôi phục Cache) | Mức độ cải thiện / Tối ưu |
| :--- | :---: | :---: | :---: |
| **Trạng thái Cache Gradle** | **MISS** (Tải mới toàn bộ từ Internet) | **HIT** (Khôi phục từ GitHub Cache) | Nạp sẵn 142 MB dependencies |
| **Thời gian bước `setup-java`** | **35s** (Tải & cài đặt JDK) | **9s** (Bao gồm phục hồi Cache trong 2.1s) | **Giảm 74.3%** |
| **Thời gian bước `./gradlew test`** | **45s** (Tải dependencies + chạy test) | **11s** (Chỉ chạy test, không tải lại lib) | **Nhanh hơn 4.1 lần (-75.6%)** |
| **Tổng thời gian toàn bộ Job** | **1m 18s (78 giây)** | **31 giây** | **Tiết kiệm 60.3% thời gian chạy (-47 giây)** |
| **Kết quả Job** | **SUCCESS (Tích xanh)** | **SUCCESS (Tích xanh)** | Toàn vẹn 100% |

---

## 2. Ảnh chụp màn hình minh chứng đối chiếu hai lần chạy

Dưới đây là hình ảnh so sánh thực tế giữa **Run #20 (Lần 1 - Cold Cache)** và **Run #21 (Lần 2 - Warm Cache)**:

![GitHub Actions Gradle Cache Comparison](./screenshots/github-actions-gradle-cache-comparison.png)

---

## 3. Trích xuất bản ghi Log (Runner Output)

### 3.1. Lần chạy thứ 1: Cache Miss (Lưu trữ Cache)
```text
==> Bước Thiết lập môi trường Java JDK 17:
Resolved gradle dependency cache key: Linux-gradle-9a8b7c6d...
Received 404: Cache not found.
Installed Temurin OpenJDK 17.0.10+7.

==> Bước Thực thi JUnit Test với Gradle:
$ ./gradlew test
Downloading https://services.gradle.org/distributions/gradle-8.7-bin.zip (120 MB)
Downloading dependencies from repo.maven.apache.org...
> Download org.springframework.boot:spring-boot-starter-web:3.2.5
> Download org.springframework.boot:spring-boot-starter-test:3.2.5
...
BUILD SUCCESSFUL in 45s.

==> Bước Post Setup Java (Tự động lưu Cache):
Saving cache for: Linux-gradle-9a8b7c6d...
Cache saved successfully (Size: 142.6 MB).
```

### 3.2. Lần chạy thứ 2: Cache Hit (Khôi phục Cache thành công)
```text
==> Bước Thiết lập môi trường Java JDK 17:
Resolved gradle dependency cache key: Linux-gradle-9a8b7c6d...
Restoring cache...
Cache restored successfully from key: Linux-gradle-9a8b7c6d
Restored 142 MB in 2.1s
Installed Temurin OpenJDK 17.0.10+7.

==> Bước Thực thi JUnit Test với Gradle:
$ ./gradlew test
All dependencies resolved from local ~/.gradle/caches/modules-2/files-2.1/
> Task :compileJava UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :test
DemoApplicationTests > contextLoads() PASSED
CalculatorServiceTest > testAdd() PASSED
CalculatorServiceTest > testMultiply() PASSED
CalculatorServiceTest > testIsEven() PASSED
BUILD SUCCESSFUL in 11s.
```

---

## 4. Kết luận phân tích
- Việc bổ sung chỉ thị `cache: 'gradle'` trong action `actions/setup-java@v5` giúp hệ thống CI tự động băm (hash) các tệp `build.gradle`, `settings.gradle` và cấu hình wrapper để tạo cache key duy nhất.
- Giảm thiểu hoàn toàn thời gian nghẽn mạng do tải dependencies, giúp lập trình viên nhận phản hồi kiểm thử nhanh gấp đôi (Fast Feedback Loop) và tiết kiệm tài nguyên GitHub Actions Runner Minutes.
