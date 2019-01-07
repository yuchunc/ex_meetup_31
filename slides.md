slidenumbers: true

# For Your Everyday Testing Needs

### Elixir & Phoenix
### \#新手友善

---

# 今天的主題

- Simple Testing
- Mocking
- Property-based Testing

(歡迎隨時補充)

---

# Simple Testing

### ~~都是騙人的~~

---

# Unit Testing

- ExUnit
- DocTest
- mix-test.watch

---

## `ExUnit`

```
defmodule FooTest do
  use ExUnit.Case

  describe "addition/2" do
    test "can add 2 numbers together" do
      assert Foo.addition(1, 1) == 2
    end
  end
end

defmodule Foo do
  def addition(x, y), do: x + y
end
```

錯誤訊息很友善

---

### Setup

```
defmodule FooTest do
  use ExUnit.Case

  setup do
    [num1: 1]
  end

  setup :iamtwo

  describe "addition/2" do
    setup _ do
      {:ok, num3: 3}
    end

    test "can add 2 numbers together", ctx do
      %{num1: n1, num2: n2, num3: n3} = ctx
      assert res = 2 = Foo.addition(n1, n2)
      assert Foo.addition(res, n3) == 6
    end
  end

  def iamtwo(_), do: %{num2: 2}
end
```

---

## `DocTest`

```
defmodule FooTest do
  doctest Foo
end

defmodule Foo do
  @doc """
  Adds 2 numbers together

  ## Examples

      iex> Foo.addition(1, 1)
      2
  """

  def addition(x, y), do: x + y
end
```

---

## mix-test.watch

- 儲存的時候跑測試
- 可以 config 你要他跑的 command
- 接受絕大部分的 `mix test` 的 flag

---

# (Pseudo) Unit Testing

- ExUnit.CaseTemplate
- ExMachina
- 資料產生

---

## 用 `ExUnit.CaseTemplate` 來 DRY

```
defmodule Foo.DataCase do
  use ExUnit.CaseTemplate

  using do
    quote do
      alias Foo.Repo

      import Ecto
      import Ecto.Changeset
      import Ecto.Query
      import Foo.DataCase
    end
  end

  setup tags do
    :ok = Ecto.Adapters.SQL.Sandbox.checkout(Foo.Repo)

    unless tags[:async] do
      Ecto.Adapters.SQL.Sandbox.mode(Foo.Repo, {:shared, self()})
    end

    :ok
  end

  def answer_to_everything, do: 42
end
```

---

## `ExMachina` (`Ecto` 專用)

```
defmodule ZaZaar.Factory do
  use ExMachina.Ecto, repo: ZaZaar.Repo

  def user_factory, do: %User{name: "維尼"}
end

insert(:user)
insert_list(3, :user, name: "跳跳虎")
```

---

## 做假資料

- Faker
- StreamData

---

# Feature Testing

- Phoenix.ChannelTest
- Wallaby
- white-bread

---

## `Phoenix.ChannelTest`

- Everything is Process
- Message Passing

---

### example

```
{:ok, _, socket} =
  socket("user:id", %{some_assigns: 1})
  |> subscribe_and_join(RoomChannel, "room:lobby", %{"id" => 3})

ref = push(socket, "my_event", %{"some" => "data"})

assert_reply ref, :ok, reply_payload
assert_push "event_pushed", push_payload
assert_broadcast "event_broadcasted", broadcast_payload
```

---

## Browser-based Feature Testing

- Wallaby
  - concurrent browser tests
- white-bread
  - async acceptance tests

---

# Mocking
## Done Right

---

## `Mox`

- By Jose Valim
- "Explicit Contracts"
- http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/

---

```
defmodule ApiBehaviour do
  @callback blob(String.t()) :: {:ok, map}, {:error, String.t()}
end

defmodule Api do
  @behaviour ApiBehaviour

  def blob(key) do
    # makes the out going call
  end
end

defmodule ApiMock do
  @behaviour ApiBehaviour

  def blob(_), do: {:error, "function wasn't assigned"}
end

defmodule Service do
  # assigns :api per env from config
  @api Application.get_env(:app_name, :api)

  def fetch_data do
    # ...
    @api.blob
    # ...
  end
end

defmodule ServiceTest do
  import Mox

  test "fetch_data gets the data and do something with it" do
    expect(ApiMock, :blob, fn _ -> {:ok, %{"returned" => "data"}} do)
    assert # ...
  end
end
```

---

# Property-Based Testing
### 電腦比你厲害

---

## `StreamData`

- 跟它說你要的範圍
- 自動產生資料
- ExUnitProperty

---

```
require ExUnitProperties

domains = [
  "gmail.com",
  "hotmail.com",
  "yahoo.com",
]

email_generator =
  ExUnitProperties.gen all name <- StreamData.string(:alphanumeric),
                           name != "",
                           domain <- StreamData.member_of(domains) do
    name <> "@" <> domain
  end

Enum.take(StreamData.resize(email_generator, 20), 2)
#=> ["efsT6Px@hotmail.com", "swEowmk7mW0VmkJDF@yahoo.com"]
```

#### https://github.com/whatyouhide/stream_data

---

> Stay Humble, Stay Foolish

---

# Thank You!
