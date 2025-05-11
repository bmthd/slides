---
marp: true
theme: default
paginate: false
class: lead
---


<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

# コールバックchildrenでロジックの見通しをよくする

<div class="flex justify-around">
  <div style="--fw: 3;">

  ```tsx
  export default function App() {
    return (
      <Toggle>
        {(on, toggle) => (
          <div>
            <p>{on ? "ON" : "OFF"}</p>
            <button onClick={toggle}>Toggle</button>
          </div>
        )}
      </Toggle>
    )
  }
  ```

  </div>
  <div style="--fw: 3;">

  ```tsx
  const Toggle = ({
    children,
  }: {
    children: (on: boolean, toggle: () => void) => React.ReactNode
  }) => {
    const [on, setOn] = useState(false)
    return <>{children(on, () => setOn(!on))}</>
  }
  ```

  </div>
</div>

<!-- <style scoped>
code {
   font-size: 1.4rem; 
   letter-spacing: -0.05em; 
   line-height: 2em;
}
:root {
  --fw: 1;
}
/* ----- レイアウト ----- */
.flex{
  display: flex;
  gap: 1em;
}
.sa {
  justify-content: space-around;
  /* border: 8px dashed rgb(15, 166, 226);
  background-color: rgb(222, 244, 255); */
}
.sb {
  justify-content: space-between;
  /* border: 8px dashed rgb(21, 17, 255);
  background-color: rgb(222, 244, 255); */
}
.sa div,.sb div{
  margin: 0.1em;
  /* border: 8px dashed rgb(80, 177, 109);
  background-color: rgb(227, 250, 237); */
}
.fw div{
  flex: var(--fw);
  /* background-color: rgb(244, 238, 255);
  border: 8px dashed rgb(93, 0, 255); */
}
</style> -->