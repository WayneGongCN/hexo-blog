---
title: React 容器组件
date: 2018-05-28
tags: react
categories: notes
---

## 容器组件 (container component)
>A container does data fetching and then renders its corresponding sub-component. That’s it.

一个容器组件仅应该用来获取数据并渲染相应的子组件。

容器组件与所渲染的子组件的关系如下
```
StockWidgetContainer => StockWidget
TagCloudContainer => TagCloud
PartyPooperListContainer => PartyPooperList
```

### 为什么需要容器组件
一个用来显示评论的组件，在不使用容器组件的情况：
```javascript
class CommentList extends React.Component {
  this.state = { comments: [] };

  componentDidMount() {
    fetchSomeComments(comments =>
      this.setState({ comments: comments }));
  }
  render() {
    return (
      <ul>
        {this.state.comments.map(c => (
          <li>{c.body}—{c.author}</li>
        ))}
      </ul>
    );
  }
}
```
除非情况完全相同，否则`CommentList`不能被重用。

### 使用容器组件
CommentListContainer
```javascript
class CommentListContainer extends React.Component {
  state = { comments: [] };
  componentDidMount() {
    fetchSomeComments(comments =>
      this.setState({ comments: comments }));
  }
  render() {
    return <CommentList comments={this.state.comments} />;
  }
}
```

CommentList
```javascript
const CommentList = props =>
  <ul>
    {props.comments.map(c => (
      <li>{c.body}—{c.author}</li>
    ))}
  </ul>
```
使用了容器组件，有如下好处
1. 将获取数据与展示数据分离，增强代码灵活程度与可阅读性。
2. `CommentList`很容易被重用。

### 

相关文章：
- [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005)
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)