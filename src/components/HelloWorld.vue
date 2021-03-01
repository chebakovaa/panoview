<script lang="tsx">
// import { GoldenLayout, LayoutConfig, LayoutManager } from 'golden-layout';
import { GoldenLayout, LayoutConfig, LayoutManager } from 'golden-layout';
import { Component, createVNode, defineComponent, onMounted, ref, render } from 'vue';

export const isClient = typeof window !== 'undefined';
export const isDocumentReady = () => isClient && document.readyState === 'complete' && document.body != null;

export function useDocumentReady(func: () => void) {
    onMounted(() => {
        console.log(isDocumentReady());
        if (isDocumentReady()) func();
        else
            document.addEventListener('readystatechange', () => isDocumentReady() && func(), {
                passive: true,
            });
    });
}

export function useGoldenLayout(components: Record<string, Component>, config?: LayoutConfig) {
    const element = ref<HTMLElement | null>(null);
    const layout = ref<GoldenLayout | null>(null);
    const initialized = ref(false);

    const componentsTypeMap = new Map<string, Component>(Object.entries(components));

    useDocumentReady(() => {
        if (element.value == null) throw new Error('Element must be set.');
        const goldenLayout = new GoldenLayout(element.value);

        const getComponentEvent: LayoutManager.GetComponentEventHandler = (container, itemConfig) => {
            const { componentType } = itemConfig;
            if (typeof componentType !== 'string') throw new Error('Invalid component type.');

            const component = componentsTypeMap.get(componentType);
            if (component == null) throw new Error(`Component not found: '${componentType}'`);

            const node = createVNode(component);
            render(node, container.element);
        };
        goldenLayout.getComponentEvent = getComponentEvent;

        if (config != null) goldenLayout.loadLayout(config);

        // https://github.com/microsoft/TypeScript/issues/34933
        layout.value = goldenLayout as any;

        initialized.value = true;
    });

    return { element, initialized, layout };
};

const Test = defineComponent(() => <span>It Works! and do</span>);

const Layout = defineComponent(() => {
    const { element } = useGoldenLayout(
        { Test },
        {
            root: {
                type: 'row',
                content: [
                    {
                        type: 'component',
                        componentType: 'Test',
                    },
                ],
            },
        }
    );
    return () => <div ref={element} style="width: 100%; height: 75vh"></div>;
});

export default Layout;
</script>

<style scoped>
</style>
