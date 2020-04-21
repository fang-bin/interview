# AJAX

封装ajax
```javascript
function ajax (options){
  let opts = Object.assign({}, options);
  opts.type = (options.type || 'GET').toUpperCase();
  opts.dataType = options.dataType || 'json';
  const formatParams = (obj) => {
    let arr = [];
    for (const name in obj) {
      if (obj.hasOwnProperty(name)) {
        arr.push(`${encodeURIComponent(name)}=${encodeURIComponent(obj[name])}`);
      }
    }
    arr.push(`v=${Math.random().toString(16).replace('.', '')}`);
    return arr.join('&');
  }
  const params = formatParams(opts.data);
  let xhr = null;
  if (window.XMLHttpRequest){
    xhr = new XMLHttpRequest();
  }else {
    xhr = new ActiveXObject('Microsoft.XMLHTTP');
  }
  xhr.onreadystatechange = function (){
    if (xhr.readyState == 4){
      const status = xhr.status;
      if (status >= 200 && status < 300){
        opts.success && opts.success(xhr.responseText, xhr.responseXML);
      }else {
        opts.fail && opts.fail(status);
      }
    }
  }
  if (opts.type === 'GET'){
    xhr.open('GET', opts.url + '?' + params, true);
  }else if (opts.type === 'POST'){
    xhr.open('POST', opts.url, true);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.send(params);
  }
}
```