# Tablespace
## File System
- Window
    - 드라이브마다 루트가 있음
        - C:\, D:\, E:\ ...
- Unix
    - 시스템 전체에 루트가 하나
        - 마운트 구조
        - /dev/hda1, /dev/sda1, /dev/cdrom
- HDFS(HaDoop)
    - 여러 대의 시스템을 묶어서 하나의 루트로 연결

## Tablespace
데이터디렉토리 상의 개별 InnoDB 파일
- MySQL Docker image 의 datadir 은 /var/lib/mysql