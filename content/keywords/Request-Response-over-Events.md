# Request-Response over Events

用事件來實現請求-回應模式。Service 先 [[Emit]] 一個查詢 [[Event]]，再 [[Listen]] 回應 [[Event]]。這種模式在本質上仍是同步思維，只是包了一層非同步的殼，稱為本課程中的「第一種非同步方式」。

## Reference

[[7_事件匯流排的非同步通訊方式.md]]
