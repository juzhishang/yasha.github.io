title: JavaScript 给表格排序
date: 2015-12-09 16:43:14
tags:
 - dom
---
```javascript
(function(){
    var mTable=document.getElementById('table');
    var sort=function(el,index,desc){

        var mTbody=el.tBodies[0],
        mRow=mTbody.rows,
        len=mRow.length,
        maxIndex=mRow[0].cells.length-1,
        arr=[],
        i;
        for(i=0;i<len;i++){
            arr[i]=mRow[i];
        }

        arr.sort(function (a,b) {
            var res;
            if(index>maxIndex){
                return 0;
            }else{            
                res=a.cells[index].innerHTML>b.cells[index].innerHTML;
                return desc?res?-1:1:res?1:-1;
            }
        });

        for(i=0;i<len;i++){
            mTbody.appendChild(arr[i]);
        }    
    }
    sort(mTable,2);
})();
```