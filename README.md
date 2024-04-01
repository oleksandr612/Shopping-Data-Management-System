
```
var API_KEY = '';
var SPREADSHEET_ID = '';
var RESULTS_PER_PAGE = 5;

function doGet(e) {
  if (!isAuthorized(e)) {
    return buildErrorResponse('not authorized');
  }
  
  var options = {
    page: getPageParam(e),
    category: getCategoryParam(e)
  }
  
  var spreadsheet = SpreadsheetApp.openById(SPREADSHEET_ID);
  var worksheet = spreadsheet.getSheets()[0];
  var rows = worksheet.getDataRange().sort({column: 2, ascending: false}).getValues();

  var headings = rows[0].map(String.toLowerCase);
  var posts = rows.slice(1);
  
  var postsWithHeadings = addHeadings(posts, headings);
  var postsPublic = removeDrafts(postsWithHeadings);
  var postsFiltered = filter(postsPublic, options.category);
  
  var paginated = paginate(postsFiltered, options.page);
    
  return buildSuccessResponse(paginated.posts, paginated.pages);
}

function addHeadings(posts, headings) {
  return posts.map(function(postAsArray) {
    var postAsObj = {};
    
    headings.forEach(function(heading, i) {
      postAsObj[heading] = postAsArray[i];
    });
    
    return postAsObj;
  });
}

function removeDrafts(posts, category) {
  return posts.filter(function(post) {
    return post['published'] === true;
  });
}

function filter(posts, category) {
  return posts.filter(function(post) {
    if (category !== null) {
      return post['category'].toLowerCase() === category.toLowerCase();
    } else {
      return true;
    }
  });
}

function paginate(posts, page) {
  var postsCopy = posts.slice();
  var postsChunked = [];
  var postsPaginated = {
    posts: [],
    pages: {
      previous: null,
      next: null
    }
  };
  
  while (postsCopy.length > 0) {
    postsChunked.push(postsCopy.splice(0, RESULTS_PER_PAGE));
  }
  
  if (page - 1 in postsChunked) {
    postsPaginated.posts = postsChunked[page - 1];
  } else {
    postsPaginated.posts = [];
  }

  if (page > 1 && page <= postsChunked.length) {
    postsPaginated.pages.previous = page - 1;
  }
  
  if (page >= 1 && page < postsChunked.length) {
    postsPaginated.pages.next = page + 1;
  }
  
  return postsPaginated;
}

function isAuthorized(e) {
  return 'key' in e.parameters && e.parameters.key[0] === API_KEY;
}

function getPageParam(e) {
  if ('page' in e.parameters) {
    var page = parseInt(e.parameters['page'][0]);
    if (!isNaN(page) && page > 0) {
      return page;
    }
  }
  
  return 1
}

function getCategoryParam(e) {
  if ('category' in e.parameters) {
    return e.parameters['category'][0];
  }
  
  return null
}

function buildSuccessResponse(posts, pages) {
  var output = JSON.stringify({
    status: 'success',
    data: posts,
    pages: pages
  });
  
  return ContentService.createTextOutput(output).setMimeType(ContentService.MimeType.JSON);
}

function buildErrorResponse(message) {
  var output = JSON.stringify({
    status: 'error',
    message: message
  });
  
  return ContentService.createTextOutput(output).setMimeType(ContentService.MimeType.JSON);
}
```
