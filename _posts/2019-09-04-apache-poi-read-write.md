---
layout: post
title: "Apache poi 를 활용한 엑셀 처리"
date: 2019-08-20 23:26:40
image: 'https://poi.apache.org/images/project-header.png'
description: Apache poi 를 활용한 엑셀 처리
category: 'java'
tags:
- build tool
introduction: Apache POI wrapper
---

실제 서비스에서 자주 사용되지 않지만, 간혹 대용량의 엑셀파일을 핸들링 해야하는 업무가 존재합니다. ( ex. 배치, 사용자 정보 엑셀다운 등등 )

JVM 환경에서 대용량 처리시 잘못된 디자인으로 개발할 경우 OOM이 발생할 가능성이 크죠, 이때 Apache POI 를 활용하여 최적화된 메모리로 대용량 엑셀 읽기 쓰기 기능을 사용할 수 있습니다.

# POI란?

> 아파치 POI(Apache POI)는 아파치 소프트웨어 재단에서 만든 라이브러리로서 마이크로소프트 오피스 파일 포맷을 순수 자바 언어로서 읽고 쓰는 기능을 제공한다. 주로 워드, 엑셀, 파워포인트와 파일을 지원하며 최근의 오피스 포맷인 Office Open XML File Formats [1] (OOXML, 즉 xml 기반의 *.docx, *.xlsx, *.pptx 등) 이나 아웃룩, 비지오, 퍼블리셔 등으로 지원 파일 포맷을 늘려가고 있다.
- 출처 [위키백과](https://ko.wikipedia.org/wiki/아파치_POI)

# POI Wrapper

POI 활용법은 [공식 사이트](https://poi.apache.org/components/spreadsheet/quick-guide.html)에서 확인이 가능합니다.

실제 사용법을 보면 객체와 작성하거 읽어오는 위치도 관리해야하는 번거로움이 존재합니다.

조금 더 편리하게 POI를 사용하기 위해 POI Wrapper를 개발하게 되었습니다.

전체 소스는 [github](https://github.com/KimYongSung/poi-wrapper)에 존재합니다.

# POI Wrapper를 활용한 대용량 파일 읽기

PoiReader의 기본적인 컨셉은 엑셀의 row별로 Object에 매핑 후 PoiRowHandler를 호출합니다. 

```java
public class PoiReaderTest {

    private File getFile() {
        return new File("excel");
    }

    @Test
    public void 엑셀파일읽기테스트() throws Exception {

        File file = getFile();

        CellInfos cellInfos = CellInfos.newInstance();

        PoiRowHandler<TestVO> rowHandler = new PoiRowHandler<TestVO>() {
            public void handler(TestVO obj) {
                System.out.println(obj.toString());
            }
        };

        PoiReader reader = PoiReader.builder(TestVO.class)
                                    .url(ResourceUtil.getURL(getFile().getAbsolutePath() + "/streming_excel_test.xlsx"))
                                    .cellNames(cellInfos.add(CellInfo.builder()
                                                                    .fieldName("test1")
                                                                    .build())
                                                        .add(CellInfo.builder()
                                                                    .fieldName("test2")
                                                                    .build()))
                                    .singletonObject()
                                    .firstRowSkip()
                                    .poiRowHandler(rowHandler)
                                    .build();

        reader.read();
        reader.close();
    }

    @Data
    public static class TestVO {

        private String test1;

        private String test2;

        private String test3;

        @Override
        public String toString() {
            return "TestVO{" +
                    "test1='" + test1 + '\'' +
                    ", test2='" + test2 + '\'' +
                    ", test3='" + test3 + '\'' +
                    '}';
        }
    }
}
```

# POI Wrapper를 활용한 대용량 파일 쓰기

PoiWriter의 기본적인 컨셉은 개발자는 데이터 출력에만 집중하고 cell, row, sheet는 내부에서 관리( 생성, 위치 )합니다. 

cell에 데이터 추가는 addCell을 호출하며, row 이동은 nextRow를 호출합니다.

최대 row를 초과하는 내부적으로 신규 sheet를 생성하여 엑셀에 write 합니다.

```java
public class PoiWriterTest {
    @Test
    public void 스트림엑셀파일생성() throws Exception {

        // given
        File file = getFile();

        PoiWriter writer = PoiWriter.getBuilder()
                                    .setSXSSFWorkbook(100)
                                    .setOutputStream(new FileOutputStream(file.getAbsolutePath() + "/streming_excel_test.xlsx"))
                                    .build();

        writer.createSheet("testSheet");

        for (int index = 0; index < 1000; index++) {
            writer.addCell(String.valueOf(index))
                  .addCell(index)
                  .nextRow();
        }

        writer.write()
              .close();
    }
}
```

-----












