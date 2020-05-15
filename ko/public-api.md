## Storage > Block Storage > API v2 가이드

API를 사용하려면 API 엔드포인트와 토큰 등이 필요합니다. [API 사용 준비](/Compute/Compute/ko/identity-api/)를 참고하여 API 사용에 필요한 정보를 준비합니다.

블록 스토리지 API는 `volumev2` 타입 엔드포인트를 이용합니다. 정확한 엔드포인트는 토큰 발급 응답의 `serviceCatalog`를 참조합니다.

| 타입 | 리전 | 엔드포인트 |
|---|---|---|
| volumev2 | 한국(판교) 리전<br>한국(평촌) 리전<br>일본 리전 | https://kr1-api-block-storage.infrastructure.cloud.toast.com<br>https://kr2-api-block-storage.infrastructure.cloud.toast.com<br>https://jp1-api-block-storage.infrastructure.cloud.toast.com |

API 응답에 가이드에 명시되지 않은 필드가 노출될 수 있습니다. 이런 필드는 TOAST 내부 용도로 사용되며 사전 공지없이 변경될 수 있으므로 사용하지 않습니다.

## 볼륨 타입
### 볼륨 타입 목록 보기
```
GET /v2/{projectid}/types
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volume_types | Body | Array | 볼륨 타입 객체 목록 |
| volume_types.id | Body | UUID | 볼륨 타입 ID |
| volume_types.name | Body | String | 볼륨 타입 이름 |
| volume_types.os-volume-type-access:is_public | Body | Boolean | 볼륨 타입 공개 노출 여부 |
| volume_types.description | Body | String | 볼륨 타입 설명 |
| volume_types.extra_specs | Body | Object | 볼륨 타입 관련 추가 사양 정보 객체 |

<details><summary>예시</summary>
<p>

```json
{
  "volume_types": [
    {
      "os-volume-type-access:is_public": true,
      "extra_specs": {
        "volume_backend_name": "ssd_general"
      },
      "id": "4e36aa51-df30-422e-aff1-eba1f3d9612f",
      "name": "General SSD",
      "description": null
    },
    {
      "os-volume-type-access:is_public": true,
      "extra_specs": {
        "volume_backend_name": "hdd_general"
      },
      "id": "6bda35e2-b2b9-497a-8f65-67a73839c856",
      "name": "General HDD",
      "description": null
    }
  ]
}
```

</p>
</details>

---

## 볼륨
### 볼륨 상태
볼륨은 다양한 상태를 가지며 상태에 따라 취할 수 있는 동작이 정해져 있습니다. 가능한 상태 목록은 다음과 같습니다.

| 상태 명 | 설명 |
|--|--|
| `creating` | 생성 중인 상태 |
| `available` | 볼륨이 생성되어 연결할 준비가 된 상태 |
| `attaching`| 볼륨이 인스턴스에 연결 중인 상태 |
| `detaching`| 볼륨이 연결 해제 중인 상태 |
| `in-use`| 볼륨이 인스턴스에 연결된 상태 |
| `maintenance`| 볼륨이 다른 호스트 장비로 이전되는 상태 |
| `deleting`| 볼륨이 삭제 중인 상태 |
| `awaiting-transfer`| 볼륨이 전송을 기다리는 상태 |
| `error`| 볼륨 생성시 오류가 발생한 상태 |
| `error_deleting`| 볼륨 삭제시 오류가 발생한 상태 |
| `backing-up`| 볼륨이 백업 중인 상태 |
| `restoring-backup`| 볼륨이 백업본으로 부터 복구되는 상태 |
| `error_backing-up`| 백업 중 오류가 발생한 상태 |
| `error_restoring`| 복구 중 오류가 발생한 상태 |
| `error_extending`| 볼륨 확장 중 오류가 발생한 상태 |
| `downloading`| 볼륨 생성시 지정한 이미지를 다운받는 상태 |
| `uploading`| 이미지 생성시 볼륨의 이미지를 업로드하는 상태 |
| `retyping`| 볼륨 타입을 변경하는 중인 상태 |
| `extending`| 볼륨을 확장하는 상태 |

### 볼륨 목록 보기
현재 테넌트에 속한 볼륨 목록을 반환합니다.

```
GET /v2/{projectId}/volumes
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| sort | Query | String | - | 정렬 기준이될 볼륨 필드 이름<br>`< key >[: < direction > ]` 형태로 기술<br>예) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 반환할 볼륨의 갯수<br>기본 값은 1000으로 설정 |
| offset | Query | Integer | - | 반환될 목록의 시작점<br>전체 목록 중 offset 번째 볼륨부터 반환 |
| marker | Query | UUID | - | 반환할 볼륨들의 직전 볼륨 ID<br>정렬 순서에 따라 `marker`로 지정된 볼 이후부터 `limit` 만큼 반환 |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volumes | Body | Array | 볼륨 객체 목록 |
| volumes.id | Body | UUID | 볼륨 ID |
| volumes.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volumes.name | Body | String | 볼륨 이름 |
| volumes_links  | Body | Object | 페이지네이션을 위한 정보 객체 (다음 목록을 가리키는 경로)<br>`limit`, `offset`를 추가한 경우 반환 |

<details><summary>예시</summary>
<p>

```json
{
  "volumes": [
    {
      "id": "90712f4f-2faa-4e4f-8eb1-9313a8595570",
      "links": [
        {
          "href": "https://kr1-api-volume.cloud.toast.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
          "rel": "self"
        },
        {
          "href": "https://kr1-api-volume.cloud.toast.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
          "rel": "bookmark"
        }
      ],
      "name": null
    }
  ]
}
```

</p>
</details>

---

### 볼륨 상세 목록 보기
현재 테넌트에 속한 볼륨 목록을 반환합니다.

```
GET /v2/{projectId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| sort | Query | String | - | 정렬 기준이될 볼륨 필드 이름<br>`< key >[: < direction > ]` 형태로 기술<br>예) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 반환할 볼륨의 갯수<br>기본 값은 1000으로 설정 |
| offset | Query | Integer | - | 반환될 목록의 시작점<br>전체 목록 중 offset 번째 볼륨부터 반환 |
| marker | Query | UUID | - | 반환할 볼륨들의 직전 볼륨 ID<br>정렬 순서에 따라 `marker`로 지정된 볼륨 이후부터 `limit` 만큼 반환 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| volumes | Body | Array | 볼륨 상세 정보 객체 목록 |
| volumes.attachments | Body | Object | 볼륨 연결 정보 객체 |
| volumes.attachments.server_id | Body | UUID | 볼륨이 연결된 인스턴스 ID |
| volumes.attachments.attachment_id | Body | UUID | 볼륨 연결 ID |
| volumes.attachments.volume_id | Body | UUID | 볼륨 ID |
| volumes.attachments.device | Body | String | 인스턴스내 장치 이름 |
| volumes.attachments.id | Body | String | 볼륨 ID |
| volumes.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volumes.availability_zone | Body | String | 볼륨 가용성 영역 |
| volumes.encrypted | Body | Boolean | 볼륨 암호화 여부 |
| volumes.os-volume-replication:extended_status | Body | String | 볼륨 확장 상태 |
| volumes.volume_type | Body | String | 볼륨 타입 이름 |
| volumes.snapshot_id | Body | UUID | 볼륨 생성시 지정한 Snapshot ID |
| volumes.id | Body | UUID | 볼륨 ID |
| volumes.size | Body | Integer | 볼륨 크기 (GB)|
| volumes.user_id | Body | String | 볼륨 소유주 ID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | 테넌트 ID |
| volumes.metadata | Body | Object | 볼륨 메타데이터 객체 |
| volumes.status | Body | Enum | 볼륨 상태 |
| volumes.description | Body | String | 볼륨 설명 |
| volumes.multiattach | Body | Boolean | 다중 연결 가능 여부<br>`true`면 여러 인스턴스에 동시에 연결할 수 있음 |
| volumes.source_volid | Body | UUID | 볼륨 생성시 지정한 Volume ID |
| volumes.consistencygroup_id | Body | UUID | 볼륨 Consistency 그룹 ID |
| volumes.name | Body | String | 볼륨 이름 |
| volumes.bootable | Body | Boolean | 볼륨 부팅 가능 여부 |
| volumes.created_at | Body | Datetime | 볼륨 생성 시각<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`의 형태 |
| volumes.os-volume-replication:driver_data | Body | String | 볼륨 복제 데이터 |
| volumes.replication_status | Body | String | 볼륨 복제 상태 |
| volumes.volumes_links  | Body | Object | 페이지네이션을 위한 정보 객체 (다음 목록을 가리키는 경로)<br>`limit`, `offset`를 추가한 경우 반환 |

<details><summary>예시</summary>
<p>

```json
{
  "volumes": [
    {
      "attachments": [],
      "links": [
        {
          "href": "https://kr1-api-volume.cloud.toast.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
          "rel": "self"
        },
        {
          "href": "https://kr1-api-volume.cloud.toast.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
          "rel": "bookmark"
        }
      ],
      "availability_zone": "kr-pub-a",
      "encrypted": false,
      "os-volume-replication:extended_status": null,
      "volume_type": "General HDD",
      "snapshot_id": null,
      "id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
      "size": 50,
      "user_id": "5e6524d826084188ae549815b3e33380",
      "os-vol-tenant-attr:tenant_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
      "metadata": {},
      "status": "available",
      "description": null,
      "multiattach": false,
      "source_volid": null,
      "consistencygroup_id": null,
      "name": "volume-f4f47065-300e-480e-8c9c-fb7ec985bffb",
      "bootable": "false",
      "created_at": "2018-12-18T05:43:12.000000",
      "os-volume-replication:driver_data": null,
      "replication_status": "disabled"
    }
  ]
}
```

</p>
</details>

---

### 볼륨 보기
지정한 볼륨에 대한 상세 정보를 반환합니다.

```
GET /v2/{projectId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| volumeId | URL | UUID | O | 볼륨 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| volume | Body | Object | 볼륨 상세 정보 객체 |
| volume.attachments | Body | Object | 볼륨 연결 정보 객체 |
| volume.attachments.server_id | Body | UUID | 볼륨이 연결된 인스턴스 ID |
| volume.attachments.attachment_id | Body | UUID | 볼륨 연결 ID |
| volume.attachments.volume_id | Body | UUID | 볼륨 ID |
| volume.attachments.device | Body | String | 인스턴스내 장치 이름 |
| volume.attachments.id | Body | String | 볼륨 ID |
| volume.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volume.availability_zone | Body | String | 볼륨 가용성 영역 |
| volume.encrypted | Body | Boolean | 볼륨 암호화 여부 |
| volume.os-volume-replication:extended_status | Body | String | 볼륨 확장 상태 |
| volume.volume_type | Body | String | 볼륨 타입 이름 |
| volume.snapshot_id | Body | UUID | 볼륨 생성시 지정한 Snapshot ID |
| volume.id | Body | UUID | 볼륨 ID |
| volume.size | Body | Integer | 볼륨 크기 (GB) |
| volume.user_id | Body | String | 볼륨 소유주 ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | 테넌트 ID |
| volume.metadata | Body | Object | 볼륨 메타데이터 객체 |
| volume.status | Body | Enum | 볼륨 상태 |
| volume.description | Body | String | 볼륨 설명 |
| volume.multiattach | Body | Boolean | 다중 연결 가능 여부<br>`true`면 여러 인스턴스에 동시에 연결할 수 있음 |
| volume.source_volid | Body | UUID | 볼륨 생성시 지정한 Volume ID |
| volume.consistencygroup_id | Body | UUID | 볼륨 Consistency 그룹 ID |
| volume.name | Body | String | 볼륨 이름 |
| volume.bootable | Body | Boolean | 볼륨 부팅 가능 여부 |
| volume.created_at | Body | Datetime | 볼륨 생성 시각<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | 볼륨 복제 데이터 |
| volume.replication_status | Body | String | 볼륨 복제 상태 |

<details><summary>예시</summary>
<p>

```json
{
  "volume": {
    "attachments": [],
    "links": [
      {
        "href": "https://kr1-api-volume.cloud.toast.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
        "rel": "self"
      },
      {
        "href": "https://kr1-api-volume.cloud.toast.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
        "rel": "bookmark"
      }
    ],
    "availability_zone": "kr-pub-a",
    "encrypted": false,
    "os-volume-replication:extended_status": null,
    "volume_type": "General HDD",
    "snapshot_id": null,
    "id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
    "size": 50,
    "user_id": "5e6524d826084188ae549815b3e33380",
    "os-vol-tenant-attr:tenant_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
    "metadata": {},
    "status": "available",
    "description": null,
    "multiattach": false,
    "source_volid": null,
    "consistencygroup_id": null,
    "name": "volume-f4f47065-300e-480e-8c9c-fb7ec985bffb",
    "bootable": "false",
    "created_at": "2018-12-18T05:43:12.000000",
    "os-volume-replication:driver_data": null,
    "replication_status": "disabled"
  }
}
```

</p>
</details>

---

### 볼륨 생성하기
스냅숏으로부터 새로운 볼륨을 생성하거나 빈 볼륨을 생성합니다.

볼륨은 생성 직후 즉시 사용할 수 없습니다. 볼륨 상태를 조회해서 `available` 상태로 전이하는 것을 확인한 후 사용합니다.

```
POST /v2/{projectId}/volumes
X-Auth-Token: {tokenId}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| volume | Body | Object | O | 볼륨 생성 요청 객체 |
| volume.size | Body | Integer | O | 볼륨 크기 (GB) |
| volume.description | Body | String | - | 볼륨 설명 |
| volume.multiattach | Body | Boolean | - | 다중 연결 가능 여부<br>`true`로 설정하면<br>여러 인스턴스에 동시에 연결할 수 있음 |
| volume.availability_zone | Body | String | - | 볼륨 가용성 영역 이름 |
| volume.name | Body | String | - | 볼륨 이름 |
| volume.volume_type | Body | String | - | 볼륨 타입 이름 |
| volume.snapshot_id | Body | UUID | - | 원본 스냅숏 ID. 생략하면 빈 볼륨이 생성됨. |
| volume.metadata | Body | Object | - | 볼륨 메타데이터 객체 |

<details><summary>예시</summary>
<p>

```json
{
    "volume": {
        "size": 10,
        "availability_zone": null,
        "source_volid": null,
        "description": null,
        "multiattach": false,
        "snapshot_id": null,
        "name": null,
        "imageRef": null,
        "volume_type": null,
        "metadata": {},
        "consistencygroup_id": null
    }
}
```

</p>
</details>

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volume | Body | Object | 볼륨 상세 정보 객체 |
| volume.attachments | Body | Object | 볼륨 연결 정보 객체 |
| volume.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volume.availability_zone | Body | String | 볼륨 가용성 영역 |
| volume.encrypted | Body | Boolean | 볼륨 암호화 여부 |
| volume.os-volume-replication:extended_status | Body | String | 볼륨 확장 상태 |
| volume.volume_type | Body | String | 볼륨 타입 이름 |
| volume.snapshot_id | Body | UUID | 볼륨 생성시 지정한 Snapshot ID |
| volumes.id | Body | UUID | 볼륨 ID |
| volume.size | Body | Integer | 볼륨 크기 (GB) |
| volume.user_id | Body | String | 볼륨 소유주 ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | 테넌트 ID |
| volume.metadata | Body | Object | 볼륨 메타데이터 객체 |
| volume.status | Body | Enum | 볼륨 상태 |
| volume.description | Body | String | 볼륨 설명 |
| volume.multiattach | Body | Boolean | 여러 인스턴스에 연결 가능 여부 |
| volume.consistencygroup_id | Body | UUID | 볼륨 Consistency 그룹 ID |
| volume.name | Body | String | 볼륨 이름 |
| volume.bootable | Body | Boolean | 볼륨 부팅 가능 여부 |
| volume.created_at | Body | Datetime | 볼륨 생성 시각<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`의 형태 |
| volume.os-volume-replication:driver_data | Body | String | 볼륨 복제 데이터 |
| volume.replication_status | Body | String | 볼륨 복제 상태 |

<details><summary>예시</summary>
<p>

```json
{
  "volume": {
    "status": "creating",
    "user_id": "94acd5b4d2bf47dda734e34a113f96ff",
    "attachments": [],
    "links": [{
      "href": "https://kr1-api-volume.cloud.toast.com/v2/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
      "rel": "self"
    }, {
      "href": "https://kr1-api-volume.cloud.toast.com/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
      "rel": "bookmark"
    }],
    "availability_zone": "kr-pub-a",
    "bootable": "false",
    "encrypted": false,
    "created_at": "2020-03-03T10:54:33.163206",
    "description": null,
    "volume_type": "General HDD",
    "name": "DATA",
    "replication_status": "disabled",
    "consistencygroup_id": null,
    "source_volid": null,
    "snapshot_id": null,
    "multiattach": false,
    "metadata": {},
    "id": "87882cf4-ca05-4ef2-b598-b93b2caf041e",
    "size": 10
  }
}
```

</p>
</details>

---

### 볼륨 삭제하기

지정한 볼륨을 삭제합니다. 연결되어 있거나 스냅샷이 생성된 볼륨은 삭제할 수 없습니다.

```
DELETE /v2/{projectId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| volumeId | URL | String | O | 볼륨 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답
이 API는 응답 본문을 반환하지 않습니다.

---

## 스냅숏
### 스냅숏 상태
스냅숏은 다양한 상태를 가지며, 상태에 따라 취할 수 있는 동작이 정해져 있습니다. 가능한 상태 목록은 다음과 같습니다.

| 상태 명 | 설명 |
|--|--|
| `creating` | 생성 중인 상태 |
| `available` | 스냅숏이 생성되어 사용할 준비가 된 상태 |
| `backing-up`| 스냅숏이 백업 중인 상태 |
| `deleting`| 스냅숏이 삭제 중인 상태 |
| `error`| 생성 중 오류가 발생한 상태 |
| `deleted`| 삭제된 상태 |
| `unmanaging`| 스냅숏에 대한 관리 모드 해제 중인 상태 |
| `restoring`| 스냅숏으로부터 볼륨을 복원 중인 상태 |
| `error_deleting`| 삭제 중 오류가 발생한 상태 |

### 스냅숏 목록 보기
스냅숏 목록을 반환합니다.

```
GET /v2/{projectId}/snapshots
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| snapshots | Body | Array | 스냅숏 정보 객체 목록 |
| snapshots.status | Body | Enum | 스냅숏 상태 |
| snapshots.description | Body | String | 스냅숏 설명 |
| snapshots.created_at | Body | Datetime | 스냅숏 생성 시간<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`의 형태 |
| snapshots.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshots.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshots.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshots.id | Body | UUID | 스냅숏 ID |
| snapshots.name | Body | String | 스냅숏 이름 |

<details><summary>예시</summary>
<p>

```json
{
  "snapshots": [
    {
      "status": "available",
      "description": "",
      "created_at": "2020-03-03T11:03:40.000000",
      "metadata": {},
      "volume_id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
      "size": 50,
      "id": "f63af601-43cd-41ab-8905-e1e93995f366",
      "name": "SNAPSHOT"
    }
  ]
}
```

</p>
</details>

---

### 스냅숏 목록 상세 보기
스냅숏 상세 정보 목록을 반환합니다.

```
GET /v2/{projectId}/snapshots/detail
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| snapshots | Body | Array | 스냅숏 상세 정보 객체 목록 |
| snapshots.status | Body | Enum | 스냅숏 상태 |
| snapshots.description | Body | String | 스냅숏 설명 |
| snapshots.os-extended-snapshot-attributes:progress | Body | String | 스냅숏 생성 진행 상태 |
| snapshots.created_at | Body | Datetime | 스냅숏 생성 시간<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`의 형태 |
| snapshots.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshots.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | 테넌트 ID |
| snapshots.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshots.id | Body | UUID | 스냅숏 ID |
| snapshots.name | Body | String | 스냅숏 이름 |

<details><summary>예시</summary>
<p>

```json
{
  "snapshots": [
    {
      "status": "available",
      "os-extended-snapshot-attributes:progress": "100%",
      "description": "",
      "created_at": "2020-03-03T11:03:40.000000",
      "metadata": {},
      "volume_id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
      "os-extended-snapshot-attributes:project_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
      "size": 50,
      "id": "f63af601-43cd-41ab-8905-e1e93995f366",
      "name": "SNAPSHOT"
    }
  ]
}
```

</p>
</details>

---

### 스냅숏 보기
지정한 스냅숏에 대한 상세 정보를 반환합니다.

```
GET /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| snapshotId | URL | UUID | O | 스냅숏 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| snapshot | Body | Object | 스냅숏 상세 정보 객체 |
| snapshot.status | Body | Enum | 스냅숏 상태 |
| snapshot.description | Body | String | 스냅숏 설명 |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | 스냅숏 생성 진행 상태 |
| snapshot.created_at | Body | Datetime | 스냅숏 생성 시간<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`의 형태 |
| snapshot.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshot.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | 테넌트 ID |
| snapshot.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshot.id | Body | UUID | 스냅숏 ID |
| snapshot.name | Body | String | 스냅숏 이름 |

<details><summary>예시</summary>
<p>

```json
{
  "snapshot": {
    "status": "available",
    "os-extended-snapshot-attributes:progress": "100%",
    "description": "",
    "created_at": "2020-03-03T11:03:40.000000",
    "metadata": {},
    "volume_id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
    "os-extended-snapshot-attributes:project_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
    "size": 50,
    "id": "f63af601-43cd-41ab-8905-e1e93995f366",
    "name": "SNAPSHOT"
  }
}
```

</p>
</details>

---

### 스냅숏 생성하기
지정한 볼륨에 대한 스냅숏을 생성합니다.

```
POST /v2/{projectId}/snapshots
X-Auth-Token: {tokenId}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| snapshot | Body | Object | O | 스냅숏 생성 요청 객체 |
| snapshot.volume_id | Body | UUID | O | 원본 볼륨 ID |
| snapshot.force | Body | Boolean | - | 강제 스냅숏 생성 여부<br>`true`이면 볼륨이 연결되어도 스냅숏을 생성 |
| snapshot.description | Body | String | - | 스냅숏 설명 |
| snapshot.name | Body | String | - | 스냅숏 이름 |

<details><summary>예시</summary>
<p>

```json
{
    "snapshot": {
        "name": "SNAPSHOT-001",
        "description": "Daily backup",
        "volume_id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
        "force": true
    }
}
```

</p>
</details>

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| snapshot | Body | Object | 스냅숏 상세 정보 객체 |
| snapshot.status | Body | Enum | 스냅숏 상태 |
| snapshot.description | Body | String | 스냅숏 설명 |
| snapshot.created_at | Body | Datetime | 스냅숏 생성 시간<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`의 형태 |
| snapshot.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshot.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshot.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshot.id | Body | UUID | 스냅숏 ID |
| snapshot.name | Body | String | 스냅숏 이름 |

<details><summary>예시</summary>
<p>

```json
{
  "snapshot": {
    "status": "creating",
    "description": null,
    "created_at": "2020-03-02T13:51:33.318695",
    "metadata": {},
    "volume_id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
    "size": 20,
    "id": "f2ae2667-b3b7-47ef-aa31-efab536888b9",
    "name": "SNAPSHOT"
  }
}
```

</p>
</details>

---

### 스냅숏 삭제하기
지정한 스냅숏을 삭제합니다.

```
DELETE /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URL | String | O | 테넌트 ID |
| snapshotId | URL | String | O | 스냅숏 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답
이 API는 응답 본문을 반환하지 않습니다.