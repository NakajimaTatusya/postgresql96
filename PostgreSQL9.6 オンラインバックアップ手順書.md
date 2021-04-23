# PostgreSQL9.6 �I�����C���o�b�N�A�b�v�菇��

* �{�h�L�������g�̍�Ƃ��s���O�����
  * �T�[�o�[OS�́AWindows Server 2016,2019��z�肵�Ă��܂��B
  * [����]�{�� PostgreSQL �T�[�o�[�Ɠ����̃X�y�b�N�̃T�[�o�[�����p�ӂ��������B
  * [�K�{]���� PostgreSQL �T�[�o�[�ցA�{�� PostgreSQL �T�[�o�[�Ɠ����o�[�W������ PostgreSQL ���C���X�g�[�����Ă����Ă��������B
  * [�K�{]�{�� PostgreSQL �T�[�o�[�ƌ��� PostgreSQL �T�[�o�[�͓����l�b�g���[�N�ɏ������A�ڑ��\�ł��鎖�����m�F���������B

* �{�h�L�������g�Ŏg�p���Ă���p��̐���

| �p�� | ���� |
| :--- | :--- |
| �{�� PostgreSQL �T�[�o�[ | ���݉^�p����Ă��� PostgresSQL9.6 �f�[�^�x�[�X�T�[�o�[ |
| ���� PostgreSQL �T�[�o�[ | �V�K�쐬���� PostgreSQL9.6 �f�[�^�x�[�X�T�[�o�[�B�o�b�N�A�b�v������B |
| pg_basebackup | PostgreSQL �r���g�C���R�}���h�B�f�[�^�x�[�X�t�@�C���̃o�b�N�A�b�v���s���B |
| pg_dump | PostgreSQL �r���g�C���R�}���h�B�_���o�b�N�A�b�v���s���B |

## �{�� PostgreSQL �T�[�o�[ �̐ݒ�

* [�K�{]�Ǘ��҃��[�U�[�̃��[�U�[���ƃp�X���[�h��p�ӂ��Ă��������B

### [�C��]���v���P�[�V�������[�U�[�쐬

```sql
-- [�C��] �����[�g�o�b�N�A�b�v���s��Ȃ��ꍇ�͕s�v�ł��B
-- �p�t�H�[�}���X�v���̂��߂ɁAREPLICATION �f�[�^�x�[�X�֐ڑ����邽�߂̃��[�U�[���쐬
-- ����ȃ��[�U�[�A���i�͎g���܂���B�p�X���[�h���ݒ肵�܂���B
CREATE USER perftest WITH REPLICATION
```

### postgresql.conf, pg_hba.conf �̊m�F

* ���L�̐ݒ�(postgresql.conf,pg_hba.conf)���s���Ă��邱�Ƃ��K�v�ł��B
* �ݒ肪�Ȃ��ꍇ�́Aconf �t�@�C����ҏW���APostgreSQL Service �� !!! �ċN�� !!! ���Ă��������B

* postgresql.conf

```ini
# [�K�{]WAL�̏o�̓��x����ݒ�A�K��l�� minimal
wal_level = replica

# [�K�{]pg_basebackup �����s���邽�߂� 1 �ȏ�ɂ���B�f�[�^�x�[�X�����v���P�[�V��������ꍇ�́Astandby�T�[�o�[���ɍ��킹�ĕύX����
max_wal_senders = 1

# [�C��] �ݒ肳��Ă��Ȃ��Ă����v�ł��B
archive_mode = 'on'
archive_command = 'copy "%p" "C:\\tmp\\archivelog\\%f"'
```

* pg_hba.conf
* ���̃t�@�C���ɂ́AREPLICATION�f�[�^�x�[�X�Ƃ�������ȃf�[�^�x�[�X�ɐڑ����邽�߂̃��[�U���`���Ă��܂��B

```ini
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

# Allow replication connections from localhost, by a user with the replication privilege.
# �{�� PostgreSQL �T�[�o�[�Ŏ��{����ꍇ
host    replication     postgres        127.0.0.1/32            md5
host    replication     postgres        ::1/128                 md5
# [�C��]���� PostgreSQL �T�[�o�[���烊���[�g�Ŏ��{����ꍇ
# trust �Ƃ��邱�ƂŃp�X���[�h�����Őڑ�
host    replication     perftest        <���� PostgreSQL �T�[�o�[��IP>/32          trust
```

## �{�� PostgreSQL �T�[�o�[�o�b�N�A�b�v���s

* �o�b�N�A�b�v�̓I�����C���o�b�N�A�b�v�ł��B�{�ԃT�[�o�[���~����K�v�͂���܂���B
* �o�b�N�A�b�v�����͒ᕉ�ׂł����A���S�̂��ߋƖ����s���Ă��Ȃ����ԑт�ݒ肷�邱�Ƃ𐄏����܂��B
* �O�̂��ߘ_���o�b�N�A�b�v�ipg_dump�j���擾���Ă���s���Ă��������B
* pg_basebackup �̎��s�́u�{�� PostgreSQL �T�[�o�[�v�ōs�����@�ƁA�u���� PostgreSQL �T�[�o�[�v���烊���[�g�ōs�����@������܂��B

### �u�{�� PostgreSQL �T�[�o�[�v�� pg_basebackup �����{���A�u���� PostgreSQL �T�[�o�[�v�փ��X�g�A����

* �o�b�N�A�b�v�p�̃f�B���N�g�����쐬����

```cmd
REM �{�� PostgreSQL �T�[�o�[�Ŏ��{����

REM YYYYMMDD ���s��
cmd> mkdir C:\tmp\PostgreSQL96\backup\YYYYMMDD
```

* pg_basebackup �����s����

```cmd
REM �{�� PostgreSQL �T�[�o�[�Ŏ��{����

REM PostgresSQL �R�}���h�̃p�X���ʂ��Ă��Ȃ��ꍇ�́A���L�փJ�����g�f�B���N�g����ύX���܂��B
cmd> cd C:\Program Files\PostgreSQL\9.6\bin

REM �I�����C���o�b�N�A�b�v���s
cmd> pg_basebackup -R -h localhost -p 5432 -U postgres -D "C:\tmp\PostgreSQL96\backup\YYYYMMDD" --xlog --checkpoint=fast --progress
```

* ���� PostgreSQL �T�[�o�[�� PostgreSQL �T�[�r�X���~����

```cmd
REM ���� PostgreSQL �T�[�o�[�Ŏ��{����

REM �T�[�r�X���� [�R���g���[�� �p�l��] > [�V�X�e���ƃZ�L�����e�B] > [�Ǘ��c�[��] > [�T�[�r�X] ����m�F���Ă��������B
REM �T�[�r�X���͌�Ђ̊����m�F���Ă��������B
net stop postgresql-x64-9.6
```

* ���� PostgreSQL �T�[�o�[�փo�b�N�A�b�v�t�@�C����S�ăR�s�[����
  * �uC:\tmp\PostgreSQL96\backup\YYYYMMDD�v ���A�u���� PostgreSQL �T�[�o�[�v�́uC:\tmp\PostgreSQL96\backup\�v�փR�s�[����

* ���� PostgreSQL �T�[�o�[�́uC:\tmp\PostgreSQL96\backup\YYYYMMDD�v�z���̐ݒ�t�@�C����ҏW����
* postgresql.conf �t�@�C����ҏW����

```ini
# postgresql.conf
hot_standby = 'off'
```

* recovery.conf �t�@�C����ҏW����

```ini
# recovery.conf
standby_mode = 'off'
primary_conninfo = 'host=localhost port=5432 user=postgres password=<password>'
recovery_target_timeline='latest'
restore_command = 'copy "C:\\tmp\\archivelog\\%f" "%p"'
```

* �_�~�[�A�[�J�C�u�t�H���_�쐬

```cmd
REM ���� PostgreSQL �T�[�o�[�Ŏ��{����

REM ���X�g�A�p�̃_�~�[�f�B���N�g�����쐬����(�A�[�J�C�u���[�h��'no'�̏ꍇ restore_command �̎w����_�~�[�t�H���_�ɂ���)
REM restore_command �p�����[�^�Ɋ֌W���܂��B

cmd> mkdir C:\tmp\archivelog

REM ��archive_mode = 'on' �ŉ^�p����Ă���ꍇ�́Aarchive_command �őޔ����Ă���WAL�A�[�J�C�u��
REM �@��L�̃t�H���_�փR�s�[���Ă����B
```

* ���� PostgreSQL �T�[�o�[�́uC:\tmp\PostgreSQL96\backup\YYYYMMDD\�v�͈ȉ��̃t�@�C���^�t�H���_��S�ăR�s�[����B

```cmd
cmd> cd C:\Program Files\PostgreSQL\9.6
cmd> xcopy C:\tmp\PostgreSQL96\backup\YYYYMMDD data /E /Y /I /D
```

* �T�[�r�X���N������

```cmd
net start postgresql-x64-9.6
```

* ������m�F����
  * [�X�^�[�g���j���[] > [PostgreSQL9.6] > [SQL Shell (psql)] �Ȃǂ��g�p���āASELECT�������s���Ă݂�

### 2��ڈȍ~�̃��X�g�A�ɂ���

* ���\���؃e�X�g���{��ɁA���� PostgreSQL �T�[�o�[��������Ԃɂ��ǂ��ɂ́A�ȉ��̎菇�����{���܂��B

* ���� PostgreSQL �T�[�o�[�� PostgreSQL �T�[�r�X���~����

```cmd
REM ���� PostgreSQL �T�[�o�[�Ŏ��{����

REM �T�[�r�X���� [�R���g���[�� �p�l��] > [�V�X�e���ƃZ�L�����e�B] > [�Ǘ��c�[��] > [�T�[�r�X] ����m�F���Ă��������B
REM �T�[�r�X���͌�Ђ̊����m�F���Ă��������B
net stop postgresql-x64-9.6
```

* ���� PostgreSQL �T�[�o�[�� PostgreSQL9.6 �C���X�g�[������̃f�[�^�x�[�X�t�H���_�ȉ��t�@�C���^�t�H���_��S�č폜

```cmd
REM PostgreSQL9.6 �C���X�g�[������̃f�[�^�x�[�X�t�H���_�ȉ��t�@�C���^�t�H���_��S�č폜
cmd> del /q "C:\Program Files\PostgreSQL\9.6\data\*"
cmd> for /d %p in ("C:\Program Files\PostgreSQL\9.6\data\*.*") do rmdir "%p" /s /q
```

* ���� PostgreSQL �T�[�o�[�́uC:\tmp\PostgreSQL96\backup\YYYYMMDD\�v�͈ȉ��̃t�@�C���^�t�H���_��S�ăR�s�[����B

```cmd
cmd> cd C:\Program Files\PostgreSQL\9.6
cmd> xcopy C:\tmp\PostgreSQL96\backup\YYYYMMDD data /E /Y /I /D
```

* �T�[�r�X���N������

```cmd
net start postgresql-x64-9.6
```

* ������m�F����
  * [�X�^�[�g���j���[] > [PostgreSQL9.6] > [SQL Shell (psql)] �Ȃǂ��g�p���āASELECT�������s���Ă݂�

### [�C��]���� PostgreSQL �T�[�o�[���烊���[�g�Ŏ��{����

* PostgresSQL �T�[�r�X���~����

```cmd
REM �T�[�r�X���� [�R���g���[�� �p�l��] > [�V�X�e���ƃZ�L�����e�B] > [�Ǘ��c�[��] > [�T�[�r�X] ����m�F���Ă��������B
REM �T�[�r�X���͌�Ђ̊����m�F���Ă��������B
net stop postgresql-x64-9.6
```

* �o�b�N�A�b�v���s
  * �V�K�쐬�����f�[�^�x�[�X�ɂȂ�܂��̂ŁA�uC:\Program Files\PostgreSQL\9.6\data�v�t�H���_�ȉ��̃t�@�C��/�t�H���_���ׂč폜���Ă��������B

```cmd
REM PostgreSQL9.6 �C���X�g�[������̃f�[�^�x�[�X�t�H���_�ȉ��t�@�C���^�t�H���_��S�č폜
cmd> del /q "C:\Program Files\PostgreSQL\9.6\data\*"
cmd> for /d %p in ("C:\Program Files\PostgreSQL\9.6\data\*.*") do rmdir "%p" /s /q
REM PostgreSQL9.6 �C���X�g�[������̃f�[�^�x�[�X�t�H���_�Ƀo�b�N�A�b�v
cmd> pg_basebackup -R -h <�{�� PostgreSQL �T�[�o�[��IP> -p 5432 -U perftest -D "C:\Program Files\PostgreSQL\9.6\data" --xlog --checkpoint=fast --progress
REM ���X�g�A�p�̃_�~�[�f�B���N�g�����쐬����(�A�[�J�C�u���[�h��'off'�̏ꍇ restore_command �̎w����_�~�[�t�H���_�ɂ���)
cmd> mkdir C:\tmp\archivelog
```

* �ݒ�t�@�C���̕ҏW

  * postgresql.conf �t�@�C����ҏW����
  * recovery.conf �t�@�C����ҏW����

```ini
# postgresql.conf
hot_standby = 'off'

# recovery.conf
standby_mode = 'off'
primary_conninfo = 'host=localhost port=5432 user=postgres password=<password>'
recovery_target_timeline='latest'
restore_command = 'copy "C:\\tmp\\archivelog\\%f" "%p"'
```

* �T�[�r�X���N������

```cmd
net start postgresql-x64-9.6
```

* ������m�F����
  * [�X�^�[�g���j���[] > [PostgreSQL9.6] > [SQL Shell (psql)] �Ȃǂ��g�p���āASELECT�������s���Ă݂�
