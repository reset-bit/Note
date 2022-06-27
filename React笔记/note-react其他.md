# react中显示markdown

```jsx
import ReactMarkdown from react-markdown;// 解析markdown
import remarkGfm from 'remark-gfm';// 解析表格、超链接等
import remarkMath from 'remark-math';// 解析数学公式等
import rehypeKatex from 'rehype-katex';// 与remark-math搭配，使用KaTeX格式解析数学公式
import MarkNav from 'markdown-navbar';// 显示目录

import 'katex/dist/katex.min.css';
import 'markdown-navbar/dist/navbar.css';

const foo = () => {
    return (
    	<>
        	<MarkNav source='demo.markdown'/>
        	<ReackMarkdown children='demo.markdown' remarkPlugins={[remarkGfm,remarkMath]} rehypePlugins={[rehypeKatex]}/>
        </>
    );
};
export default foo;
```

