## 使用 gomock 生成基于 interface 的单元测试代码

[gomock: https://github.com/golang/mock](https://github.com/golang/mock) 是 Golang 官方提供的 mock 框架。

### 1. 安装

```bash
# get gomock
go get -u github.com/golang/mock/gomock
# 获取 用于生成辅助代码的 mockgen
go get -u github.com/golang/mock/mockgen
```

### 2. 生成辅助代码

假设有一个项目`demo`，其项目结构如下：

```bash
demo
├── main.go
└── pkg
    ├── model
    │   └── age.go
    └── service
        └── service.go
```

其中 `age.go` 中定义了一个 `interface`:

```go
type Age interface {
    GetCount(ctx context.Context, id string)(int32,error)
}
```

数据处理交给 `model` 层，具体怎么实现需要按照实际情况。在 `service` 层我们并不关心 `model` 是如何实现的。我们只需要调用对应的 `interface` 获取数据即可。现在我们要对 `service` 层编写 `service_test`，而具体的 `model` 的实现有时直接影响了 `service` 层的处理，因此，我们需要对 `model` 层进行 `mock`。

使用 `mockgen` 工具生成 `model` 层的 `mock`辅助代码：

```bash
# 进入到 interface 所在的目录
cd ~/demo/pkg/model
# 生成 mock 代码
mockgen -destination mock/age_mock.go -package mock -source age.go
```

生成后目录结构如下：

```bash
demo
├── main.go
└── pkg
    ├── model
    │   ├── age.go
    │   └── mock
    │       └── age_mock.go
    └── service
        └── service.go
```

### 3. 编写 service_test

假设我们 `service.go` 中对外暴露了一个接口，用户获取用户信息，其中 `model` 中的 `Age interface` 用于获取用户年龄：

```go
package service

import (
	"context"
	"demo/pkg/model"
)

type MyService struct {
	ageModel model.Age
}

func NewMyService(am model.Age) *MyService {
	return &MyService{ageModel: am}
}

func (inv *MyService) GetUserInfo(ctx context.Context, id string) (*User, error) {
	// .....
	age, err := inv.ageModel.GetAge(ctx, id)
	if err != nil {
		//...
	}
	// ...

    return &User{Age: age}, nil
}

```

我们在 `service.go`  的同级目录中新建 `service_test.go`：

```go
package service

import (
	"context"
	"demo/pkg/model/mock"
	"errors"
	"testing"

	"github.com/golang/mock/gomock"
	"github.com/stretchr/testify/assert"
)

func TestMyService_GetUserInfo(t *testing.T) {
	ctrl := gomock.NewController(t)
	defer ctrl.Finish() // 确保 Age 中的方法被调用

	ageModel := mock.NewMockAge(ctrl)
	ctx := context.Background()

	// 当id="id:123"时，确切返回 (123,nil)
	ageModel.EXPECT().GetAge(ctx, gomock.Eq("id:123")).Return(123, nil)

	// 当 id != "id:123"，返回 (0, errors.New("not found"))
	ageModel.EXPECT().GetAge(ctx, gomock.Not("id:123")).Return(0, errors.New("not found"))

	// 任一参数，都返回 (111, nil)
	ageModel.EXPECT().GetAge(ctx, gomock.Any()).Return(111, nil)

	// 输入的事 Nil 值，返回 (222, nil)
	ageModel.EXPECT().GetAge(ctx, gomock.Nil()).Return(222, nil)
    
    // 这些逻辑也可以放在一个函数中 
    ageModel.EXPECT().GetAge(gomock.Any()).DoAndReturn(func(ctx context.Context, id string)(int32, error){
        if id == "id:123" {
            return 123, nil
        }
        return 0, errors.New("not found")
    })

	// 进行接口测试
	res, err := NewMyService(ageModel).GetUserInfo(ctx, "id:123")
	assert.Nil(t, err)
	assert.Equal(t, res.User.Age, 123)
}
```

可以看到，在编写 `service` 层的接口测试时，`mock.MockAge` 实现了 `model.Age` 接口，我们可以直接将其实例赋值给 `MyService`。

## 使用 mockery 生成基于 grpc 的 ServiceClient mock 代码

[https://github.com/vektra/mockery](https://github.com/vektra/mockery) 是一个第三方库，也用于生成基于 interface 的 mock 代码。

假设我们要调用另一个服务： `inventory-store`，服务间使用 `grpc` 调用，`proto文件位于` `pb/inventory.proto`， 里面定义了 `GetProduct` 接口：

```proto
syntax = "proto3";
package pb;

service InventoryStoreService {
    rpc GetProduct(GetProductReq) returns(GetProductReply){};
}

message GetProductReq {
    string id = 1;
}
message GetProductReply {
    string product_name = 1;
    int32 left_count = 2;
}
```

### 第一步：安装 mockery:

```bash
brew install mockery
```

### 第二步，进入到 `pb` 目录，执行以下命令，生成对应的 mock 代码(假设已经生成了对应的 inventory-store.pb.go):

```bash
mockery --name=InventoryStoreSrvClient --keeptree --output=. --filename=mock_InventoryStoreSrvClient.go
```

### 第三步，使用 mock 代码模拟远程调用：

用法参考 [README.md](https://github.com/vektra/mockery/blob/master/README.md) 。