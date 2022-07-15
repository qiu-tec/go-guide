# Gorm使用规范



## 1、数据库访问层规范

​	以表为单位，分为表实体entity和表访问dal两层

​	以样本表（Sample）为例

​	目录结构

```
├── db
│   └── entity
│       └── sample.go
│       └── ...
│   └── dal
│       └── sampleDal.go
│       └── ...
│ ...
```



## 2、开发阶段请打开sql输出

```go
	db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{
         // 打开脚本输出
		Logger: logger.Default.LogMode(logger.Info),
	})
```



## 3、带自增主键的实体定义规范

```go
// 统一按Id命名，类型为uint32，并增加 `gorm:"primaryKey"` 标记

type Sample struct {
	Id     uint32 `gorm:"primaryKey"`
	Date   uint   `gorm:"index"`
	Field1 string
	Field2 string
}
```



## 4、Insert

插入一条数据

- 传入：实体指针
- 返回：执行状态，异常信息

```go
func (dal *SampleDal) Insert(sample *entity.Sample) (bool, error) {
	result := dal.db.Create(sample)
	if result.RowsAffected > 0 {
		return true, nil
	}
	return false, result.Error
}

// Insert(&sample)
// INSERT INTO `samples` (`date`,`field1`,`field2`) VALUES (0,"","") RETURNING `id`

// 插入完成后，如何获取自增的值
sample := Sample{}
ok, err := d.Insert(&sample)
if ok {
    fmt.PrintIn("写入后的自增编号为：", model.Id)
}

```



## 5、Update

更新一条数据

- 传入：条件，需要更新的内容（其中key为表字段，value为需要更新的值）
- 返回：执行状态，异常信息

```go
func (dal *SampleDal) Update(id uint, value map[string]interface{}) (bool, error) {
	// 格式为：db.Model(空表实体).Where("条件",...).Updates(需要修改的字段)
	result := dal.db.Model(&entity.Sample{}).Where("id = ?", id).Updates(value)
	if result.RowsAffected > 0 {
		return true, nil
	}
	return false, result.Error
}

// Update(1, map[string]interface{}{"field1": "11", "field2": ""})
// 生成SQL：UPDATE `samples` SET `field1`="11",`field2`="" WHERE id = 1
```



## 6、Delete

删除一条数据

- 传入：条件
- 返回：执行状态，异常信息

```go
func (dal *SampleDal) Delete(id uint) (bool, error) {
	// 格式：db.Where("条件",...).Delete(空表实体)
	result := dal.db.Where("id = ?", id).Delete(&entity.Sample{})
	if result.RowsAffected > 0 {
		return true, nil
	}
	return false, result.Error
}

// Delete(1)
// 生成SQL：DELETE FROM `samples` WHERE id = 1
```



## 7、GetModel

获取一条数据

- 传入：条件
- 返回：结构体，异常信息

```go
func (dal *SampleDal) GetModel(id uint) (*entity.Sample, error) {
	sample := &entity.Sample{}
	result := dal.db.Where("id = ?", id).Find(sample)
	// 如果异常或者未查询到任何数据
	if result.RowsAffected == 0 || result.Error != nil {
		return nil, result.Error
	}
	return sample, nil
}

// GetModel(1)
// 生成SQL：SELECT * FROM `samples` WHERE id = 1
```



## 8、GetList

获取一组数据

- 传入：条件
- 返回：结构体列表，异常信息

```go
func (dal *SampleDal) GetList(date uint) ([]entity.Sample, error) {
	list := make([]entity.Sample, 0)
	result := dal.db.Where("date = ?", date).Find(&list)
	return list, result.Error
}

// GetList(1)
// 生成SQL：SELECT * FROM `samples` WHERE date = 0
```



## 9、Order用法

获取一组数据，并按指定字段排序

- 传入：条件
- 返回：结构体列表，异常信息

```go
func (dal *SampleDal) GetListByOrder(date uint) ([]entity.Sample, error) {
	list := make([]entity.Sample, 0)
	// 如果需要多个字段排序，使用逗号分割
	// 例如 date desc, id asc
	result := dal.db.Where("date = ?", date).Order("id desc").Find(&list)
	return list, result.Error
}

// 生成SQL：SELECT * FROM `samples` WHERE date = 0 ORDER BY id desc
```



## 10、Limit用法

获取一组列表，按ID逆序，并返回前5行

- 传入：条件
- 返回：结构体列表，异常信息

```go
func (dal *SampleDal) GetListByLimit(date uint) ([]entity.Sample, error) {
	list := make([]entity.Sample, 0)
	result := dal.db.Where("date = ?", date).Order("id desc").Limit(5).Find(&list)
	return list, result.Error
}

// 生成SQL：SELECT * FROM `samples` WHERE date = 0 ORDER BY id desc LIMIT 5
```

