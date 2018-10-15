查询结构
GET call_data*/_mapping

模糊查询
GET  /call_data*/dial_result/_search
{
  "query": {
    "wildcard": {
      "phone":"*3857"
    }
  }
}